# Intermediate Solidity
This content is to showcase some Solidity patterns. Corrections and additional requests are welcomed.

## Contents
* [Sequence of actions](#sequence-of-actions)
* [Organizing data](#organizing-data)
* [Mutexes protect from re-entrancy](#mutexes-protect-from-re-entrancy)
* [Group of admins](#group-of-admins)

## Sequence of actions 
### a.k.a. "Checks-Effects-Interactions Pattern"
A security recommendation, that is the basic pattern in Solidity and should be carefully respected.
See: https://solidity.readthedocs.io/en/develop/security-considerations.html?#use-the-checks-effects-interactions-pattern
There be dragons in external contracts that we might be calling! There be dangerous beasts when the atomicity breaks or the control flow runs amok, yet we be stuck in an immutable contract.

To be on the safe side, write functions by following these steps in strict order:
1. **Check** the parameters of the function invocation: arguments, balance and other context.
1. **Update** the state (to prevent possible re-entrancy).
1. **Log** the changes to alert the off-chain app and to make the history reconstruction easier.
1. **Call** external contracts, if needed. Try not to.
1. **Return** outputs, if needed. Consider that outputs make the caller aware of the completion/success of his call.
```
pragma solidity ^0.5.0;

contract CEI_hole {
    uint public counter;
    
    function increaseCounter() public returns (bool) {
        counter += 1;
        //do evil here ... and return true as if everything is nice
        return true;
    }
}

contract ChecksEffectsInteractions {
    bool status;
    CEI_hole c;
    
    event LogNice(bool newStatus, uint atN);
    
    constructor(address _extAdr) public { c = CEI_hole(_extAdr); }
    
    function niceF(uint _n) public returns (uint) {
        bool extRes;

        require(_n <= 42, "n not nice!"); //1. check
        status = true;                 //2. update
        emit LogNice(status, _n);      //3. log
        extRes = c.increaseCounter();  //4. call
        if (extRes) return 1;          //5. return
        else return 0;
    }
}
```

## Organizing data
Structuring data is inevitably going to take some of our time setting up to suit our needs. Solidity comes with only a few building blocks: structs to create custom types,  which we can then collect into arrays or mappings.
See also in-depth posts by Rob Hitchens here: https://medium.com/@robhitchens/solidity-crud-part-1-824ffa69509a

Arrays can be fixed-size or dynamic, provide indexed data, but we can't access data by keys which means we need to search sequentially - which means that after a certain (unknown) number of users, the costs would be prohibitively high and our contract would stop working with out-of-gas exception! We also can't check for duplicates.
```
pragma solidity ^0.5.0;

contract DataArrays {
    struct User {
        uint age;
        address addr;
    }
    
    User[] public users;
    uint public arrLen;
        
    function arrays() public {
        User memory u;
        u.age = 13;
        u.addr = msg.sender;
        users.push(u);
        
        u.age = 42;
        users.push(u);
        
        //this just resets the struct to its default values!
        delete users[0];
        arrLen = users.length; //so length is still 2
        
        //finding a particular user is expensive and can break with OOG!
        for (uint i; i < users.length; i++) {
            if (users[i].age == 42) {
                //do something, then exit the loop
                break;
            }
        }
    }
}
```

Mappings are a very useful type, that maps the keys to their values, where keys can be basically any simple type and values can be almost anything. Mappings protect us from duplicate keys. But now we can't access data by keys, we can't count the elements, we can't iterate.

Note that mappings are allowed for state variables only. They are implemented without iterators, so you can’t get the next or previous value. Actually, you can’t even get a key out of it, since only SHA3(key[i]) is stored, which is in turn used to look up the corresponding value.

To determine whether an element exists, as opposed to being un-initialized and having default values, we can extend the basic struct.
In general, it can be tricky to differentiate from a simple query between getting a non-existing element or a valid 0 (or 0x0 or other default value). We can throw an exception if the element is not found!

```
pragma solidity ^0.5.0;

contract DataMapping {
    struct User {
        string name;
        uint age;
        bool exists;
    }
    
    mapping(address => User) public users;
    uint public dummyAge;
    
    function existsEl(address _key) public view returns (bool) {
        return users[_key].exists; //we can check whether the element exists 
    }
    
    function addEl(address _addrUser, string memory _newName, uint _newAge) public {
        users[_addrUser] = User({
            name: _newName,
            age: _newAge,
            exists: true
        });
    }
    
    function getElAge(address _key) public view returns (uint) {
        require(existsEl(_key), "Doesn't exist.");
        return users[_key].age; //we can immediately access values of a random key
    }
    
    function deleteEl(address _key) public {
        require(existsEl(_key), "Doesn't exist.");
        users[_key].exists = false; //delete could mean just setting it as non-existent
    }
        
    function mappings(address _key) public {
        addEl(msg.sender, "Young", 13);
        addEl(msg.sender, "Old", 42); //value gets overwritten, because current addEl doesn't check for existence
        dummyAge = getElAge(_key); //42
        deleteEl(_key);
    }
    //we can't find the number of users!
}
```
Bringing the two together, using a mapping + an array, we get an indexed mapping, a.k.a. an iterable mapping.
*In the first attempt to overcome the previous challenges, we almost make it, but the delete functionality now breaks - if you don't need to delete elements, than it is ok, otherwise scroll down to the next snippet.*
```
pragma solidity ^0.5.0;

contract DataMappingNoDelete {
    struct User {
        string name;
        uint age;
        bool exists;
    }
    
    mapping(address => User) public users;
    address[] public userKeys;
    
    uint public dummyAge;
    uint public usrCount;
    
    function existsEl(address _key) public view returns (bool) {
        return users[_key].exists; 
    }
    
    function addEl(address _addrUser, string memory _newName, uint _newAge) public {
        users[_addrUser] = User({
            name: _newName,
            age: _newAge,
            exists: true
        });
        
        userKeys.push(_addrUser); //add the new key to the array of keys
    }
    
    function getElAge(address _key) public view returns (uint) {
        require(existsEl(_key), "Doesn't exist.");
        return users[_key].age;
    }
    
    function deleteEl(address _key) public {
        require(existsEl(_key), "Doesn't exist.");
        users[_key].exists = false;
    }
        
    function mappings(address _key) public {
        addEl(msg.sender, "Young", 13);
        addEl(msg.sender, "Old", 42);
        dummyAge = getElAge(_key);
        usrCount = userKeys.length; //we can find the number of users
        deleteEl(_key); //NOT WORKING CORRECTLY!
        usrCount = userKeys.length; //returns 2 - the delete is breaking our count!
    }
}
```
To support delete operations, we extend the basic struct with an index, which will serve to replace the deleted element with the last one in the array (think pointers) and finally pop it from the array. 
Note that just deleting the last element won't be enough, because delete just resets the values to defaults, thus we need to reduce the size of the array instead.
Note also that `push` [returns the length of the new array](https://solidity.readthedocs.io/en/latest/types.html#members), which we could directly use to specify the index of the newly inserted element. We omit it here for readability.
```
pragma solidity ^0.5.0;

contract DataMapping {
    struct User {
        string name;
        uint age;
        uint idx;
    }
    
    mapping(address => User) public users;
    address[] public userKeys;
    
    uint public dummyAge;
    uint public usrCount1;
    uint public usrCount2;
    
    function existsEl(address _key) public view returns (bool) {
        bool result;
        uint tmpIndex;
        if (userKeys.length == 0) return false;
        
        tmpIndex = users[_key].idx;
        if (userKeys[tmpIndex] == _key) result = true;
        return result;
        //or in one line: return (userKeys[users[_key].idx] == _key);
    }
    
    function getIdx(address _key) public view returns (uint) {
        require(existsEl(_key), "Doesn't exist.");
        return users[_key].idx;        
    }    
    
    function addEl(address _addrUser, string memory _newName, uint _newAge) public {
        users[_addrUser] = User({
            name: _newName,
            age: _newAge,
            idx: userKeys.length
        });        
        userKeys.push(_addrUser); //add the new key to the array of keys
    }
    
    function getElAge(address _key) public view returns (uint) {
        require(existsEl(_key), "Doesn't exist.");
        return users[_key].age;
    }
    
    function deleteEl(address _key) public {
        uint idx2Del;
        address key2Move;
        
        idx2Del = getIdx(_key); //find the index of the element we are deleting
        key2Move = userKeys[userKeys.length-1]; //find the address of the last element in the array
        
        userKeys[idx2Del] = key2Move;  //move pointer at the current index to the last element
        users[key2Move].idx = idx2Del; //update the pointer in the last element to the current index
        
        userKeys.length--; //drop the last element and reduce array size
    }
        
    function mappings(address _key) public {
        addEl(0xCA35b7d915458EF540aDe6068dFe2F44E8fa733c, "Young", 13);
        addEl(0x14723A09ACff6D2A60DcdF7aA4AFf308FDDC160C, "Old", 42);
        dummyAge = getElAge(_key);
        usrCount1 = userKeys.length; //=2 ... we can find the number of users
        deleteEl(_key);             //finally works ok
        usrCount2 = userKeys.length; //=1
    }
}
```
As a final note, try making changes to basic data structures *private*, so that we limit the exposure to modifications. Be sure to update as an atomic transaction - either update both the mapping *and* the array or none of them.



## Mutexes protect from re-entrancy
Making sure that the control flow is as we expect it to be can be hard. Say that we want one function to execute in it entirety, not allowing nested calls that could for example drain our contract. We could do it via mutex, that allows execution only when the locking flag is not raised - we raise the flag in the first step of the actual execution.
See also a more efficient Re-entrancy Guard contract by OpenZeppelin: https://github.com/OpenZeppelin/openzeppelin-solidity/blob/master/contracts/utils/ReentrancyGuard.sol
```
pragma solidity ^0.5.0;

contract NaiveContract {
    uint public balance;
    bool private lock;
    
    modifier Mutex() {
        require(!lock);
        lock = true;
        _;
        lock = false;
    }
    
    function deposit() payable public {
        balance += msg.value;
    }
    
    function withdrawHalf() public {
        uint halfBalance = address(this).balance/2;
        bool withdrawResult;
        bytes memory returning;
        
        require(!lock);
        lock = true; //comment this to disable the mutex and make this contract vulnerable
        
        (withdrawResult, returning) = msg.sender.call.value(halfBalance)("");
        if (withdrawResult) {
            balance = halfBalance;
        }
        lock = false;
    }

    function getBalance() public view returns (uint) {
        return address(this).balance;
    }    
}

contract Attacker {
    NaiveContract n;
    
    constructor(address _victim) public {
        n = NaiveContract(_victim);
    }
    
    function withdrawAll() public {
        n.withdrawHalf();
    }
    
    function() external payable {
       n.withdrawHalf(); //upon receiving the first half, this fallback will trigger yet anothe withdrawall ...
    }
    
    function getBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```

## Group of admins
We often want to limit certain functionalities of our contracts to administrators, which is mostly done via `ownerOnly` modifiers. Since that in essence presents a SPOF, we might want to instead allow these special functionalities to m-of-n administrators.

To implement such an idea, we need a collection of special functionalities with corresponding thresholds and a way to accumulate administrators' votes.
An alternative and cheaper way would be to make admin transactions from a multi-signature protected account, where an admin transaction would require m-of-n signatures to be prepared off-chain.

Example: [pausing a contract](https://consensys.github.io/smart-contract-best-practices/software_engineering/#circuit-breakers-pause-contract-functionality) in an emergency, where 2-of-3 admins need to comply for changing the stage. Mind the simplifications for the sake of brevity.
```
pragma solidity ^0.5.0;

contract MultiOwn {
    bool public isPaused;
    
    //for simplicity, we assume only two special operations: one to pause and one to un-pause the contract
    bytes8 public constant B_PAUSE_ON = "\x70\x61\x75\x73\x65\x4f\x6e"; //bytes8(pauseOn) = 0x70617573654f6e
    bytes8 public constant B_PAUSE_OFF = "\x70\x61\x75\x73\x65\x4f\x66\x66"; //bytes8(pauseOff)= 0x70617573654f6666
    
    address[3] public owners; //for simplicity, we assume 3 owners
    mapping(bytes8 => uint) public opEndorsements;
    mapping(address => mapping(bytes8 => bool)) public ownersEndorsements;
    
    event LogPause(bool isPaused, uint threshold);
    
    constructor(address[3] memory _owners) public { for (uint i; i < _owners.length; i++) { owners[i] = _owners[i]; } }    
    
    modifier notPaused() { require(!isPaused); _; }
    
    modifier ownerOnly() {
        bool senderIsOwner;
        for (uint8 i; i < owners.length; i++) { if (msg.sender == owners[i]) { senderIsOwner = true; break; } }
        require(senderIsOwner, "Not owner.");
        _;
    }
    
    //for simplicity, 2-of-3 threshold
    modifier threshOnly(bytes8 _op) {
        if (_op == B_PAUSE_ON) require(opEndorsements[B_PAUSE_ON] >= 2); 
        else if (_op == B_PAUSE_OFF) require(opEndorsements[B_PAUSE_OFF] >= 2);
        _;
    }
    
    
    //protected by a threshold of admins
    function pauseOn() external threshOnly(B_PAUSE_ON) { 
        isPaused = true; 
        emit LogPause(isPaused, opEndorsements[B_PAUSE_ON]);
        //reset endorsements for this particular operation
        delete opEndorsements[B_PAUSE_ON];
        for (uint8 i; i < owners.length; i++) {
            delete ownersEndorsements[owners[i]][B_PAUSE_ON];
        }
    }
    
    //protected by a threshold of admins    
    function pauseOff() external threshOnly(B_PAUSE_OFF) {
        isPaused = false;
        emit LogPause(isPaused, opEndorsements[B_PAUSE_OFF]);
        //reset endorsements for this particular operation
        delete opEndorsements[B_PAUSE_OFF];
        for (uint8 i; i < owners.length; i++) {
            delete ownersEndorsements[owners[i]][B_PAUSE_OFF];
        }        
    }
    
    //collect endorsements against the threshold of a particular operation
    function endorseOp(bytes8 _op) external ownerOnly {
        require(!ownersEndorsements[msg.sender][_op], "Already endorsed.");
        
        opEndorsements[_op]++;
        ownersEndorsements[msg.sender][_op] = true;
    }
    
    
    //finally ...
    function specialFunction() external notPaused {
        //do something delicate, that could be paused by m-of-n owners
    }
}
```


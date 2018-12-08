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
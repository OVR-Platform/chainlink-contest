<img src="logo.svg" style="height: 90px; margin-bottom: 30px;" /> 

# OVR Chainlink Competition

Visit detailed rules on our [website](https://www.ovr.ai/blog/ovr-competition-powered-by-chainlink/).


## Smart Contract Source Code

Randomness is very difficult to generate on blockchains. The reason for this is because every node must come to the same conclusion, forming a consensus. There's no way to generate random numbers natively in smart contracts, which is unfortunate because they can be very useful for a wide range of applications.


Chainlink VRF (Verifiable Random Function) is a provably-fair and verifiable source of randomness designed for smart contracts. Smart contract developers can use Chainlink VRF as a tamper-proof RNG to build reliable smart contracts for any applications which rely on unpredictable outcomes:

- Blockchain games and NFTs
- Random assignment of duties and resources (e.g. randomly assigning judges to cases)
- Choosing a representative sample for consensus mechanisms

Chainlink VRF enables smart contracts to access randomness without compromising on security or usability. With every new request for randomness, Chainlink VRF generates a random number and cryptographic proof of how that number was determined. The proof is published and verified on-chain before it can be used by any consuming applications. This process ensures that the results cannot be tampered with nor manipulated by anyone, including oracle operators, miners, users and even smart contract developers.

Here how OVR uses Chainlink [VRFConsumerBase](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.6/VRFConsumerBase.sol) Contract:

```solidity
pragma solidity 0.6.6;

import "@chainlink/contracts/src/v0.6/VRFConsumerBase.sol";
import "@chainlink/contracts/src/v0.6/Owned.sol";

contract OVRChainlinkCompetition is VRFConsumerBase, Owned  {
    bytes32 internal keyHash;
    uint256 internal fee;

    uint256 public randomResult;
    uint public times = 0;

     /**
     * Constructor inherits VRFConsumerBase
     * 
     * Network: Ethereum Mainnet
     * Chainlink VRF Coordinator address: 0xf0d54349aDdcf704F77AE15b96510dEA15cb7952
     * LINK token address:                0x514910771AF9Ca656af840dff83E8264EcF986CA
     * Key Hash: 0xAA77729D3466CA35AE8D28B3BBAC7CC36A5031EFDC430821C02BC31A238AF445
     */
    constructor() 
        VRFConsumerBase(
            0xdD3782915140c8f3b190B5D67eAc6dc5760C46E9, // VRF Coordinator
            0xa36085F69e2889c224210F603D836748e7dC0088  // LINK Token
        ) public {
        keyHash = 0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4;
        fee = 2 * 10 ** 18; 
        // 2 LINK (Ethereum Mainnet)
    }

    modifier onlyOneTime() {
        require(times == 0, "You can ask it only one time.");
        _;
    }

    /** 
     * Requests randomness 
     */
    function getRandomNumber() public onlyOwner onlyOneTime returns (bytes32 requestId) {
        require(LINK.balanceOf(address(this)) >= fee, "Not enough LINK");
        times += 1;
        return requestRandomness(keyHash, fee);
    }

    /**
     * Callback function used by VRF Coordinator
     * Random Number between 1 and NUMBER_OF_PARTICIPANTS.
     * +1 because the random number can be zero
     */
    function fulfillRandomness(bytes32 requestId, uint256 randomness) internal override {
        randomResult = (randomness % NUMBER_OF_PARTICIPANTS) + 1;
    } 

    receive() external payable {

    }
}
```

The contract is deployed on the Ethereum blockchain for each competition and gives the ability for the deployer alone to launch the getRandomNumber() function which can only be called once.
The contract remains the same, in each competition the only variable is the number of participants.



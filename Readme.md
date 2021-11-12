## The three contract we are interested in are: 

**[Comp](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/Comp.sol)**

**[GovernorAlpha](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Governance/GovernorAlpha.sol)**

**[Timelock](https://github.com/compound-finance/compound-protocol/blob/master/contracts/Timelock.sol)**


## Create your project

**Requirements:** 

You will need to have `node` installed on your machine. We will be using [HardHat](https://hardhat.org/getting-started/) for our scripting and deployment needs. 

**Get Started:**

First we are going to create a directory for our project: 

`mkdir tutorial-governance` 

then CD into it: 

`cd tutorial-governance`

We will want to initialize our node project: 

`npm init -y`

The "-y" will pre-populate our `.json` file. 

Now we want to setup our HardHat project: 

`npx hardhat`

This will setup our initial HardHat Project with some defaults.

- Create a sample Project
- Accept the project root
- Accept creating a .gitignore
- Accept installing the sample project dependencies

At this point you might want to spend a couple minutes inspecting the HardHat setup if this is your first time working with it. 


### Create the files

We will be creating three files in our `/contracts` folder: `COMP.sol` `Timelock.sol` `GovernorAlpha.sol`. You can take these files directly from the Compound Github or copy from the verified code in Etherescan. 

There will be a leftover file from the HardHat install called `Greeter.sol` in the contracts directory. You can delete this. 

When you are finished your `/contracts` folder should have three contracts in it. 

```markdown
/Contracts
	COMP.sol
	Timelock.sol
	GovernorAlpha.sol
```

Easy!

**Customizing Parameters**

A number of parameters in the Compound system are hard coded, these parameters you will most likely want to customize for your installation: 

### **COMP**

```jsx
contract Comp {
    /// @notice EIP-20 token name for this token
    string public constant name = "MyTokenName";

    /// @notice EIP-20 token symbol for this token
    string public constant symbol = "MTN";

    /// @notice EIP-20 token decimals for this token
    uint8 public constant decimals = 18;

    /// @notice Total number of tokens in circulation
    uint public constant totalSupply = 10000000e18; // 10 million myTokens
```

In the COMP contract the constants we might want to alter are right at the top of the file: 

`name`

`symbol`

`decimals` 

`totalSupply`

These are standard ERC20 properties, the Name, and Symbol (for exchanges that list the token), the decimals, and total supply. In this case, you don't want to alter the `decimals`, the ecosystem has made `18` the standard, and using a different number can cause problems or incompatibilities with the rest of the ERC20 ecosystem. 

Feel free to change `name`, `symbol` and `totalSupply` to whatever you like and run `npx hardhat compile` once again. 

### **GovernorAlpha.sol**

The next step is to customize `GovernorAlpha` contract. At the top of the source code file: `/contracts/GovernorAlpha.sol` that you created earlier with the copy-paste code from etherscan you should see the following: 

```jsx
contract GovernorAlpha {
    /// @notice The name of this contract
    string public constant name = "Compound Governor Alpha";

    /// @notice The number of votes in support of a proposal required in order for a quorum to be reached and for a vote to succeed
    function quorumVotes() public pure returns (uint) { return 400000e18; } // 400,000 = 4% of Comp

    /// @notice The number of votes required in order for a voter to become a proposer
    function proposalThreshold() public pure returns (uint) { return 100000e18; } // 100,000 = 1% of Comp

    /// @notice The maximum number of actions that can be included in a proposal
    function proposalMaxOperations() public pure returns (uint) { return 10; } // 10 actions

    /// @notice The delay before voting on a proposal may take place, once proposed
    function votingDelay() public pure returns (uint) { return 1; } // 1 block

    /// @notice The duration of voting on a proposal, in blocks
    function votingPeriod() public pure returns (uint) { return 17280; } // ~3 days in blocks (assuming 15s blocks)
```

Here the items that interest us for customization are: 

`name` 

`quorumVotes()` 

`proposalThreshold()`

`proposalMaxOperations()`

`votingDelay()`

`votingPeriod()`

**Name**: This self explanatory, change it to whatever you would like. 

**QuorumVotes (`quorumVotes()`)** 

The quorum is the number of **YES** votes required to ensure that a vote is valid. The idea behind this is that some minimum number of votes need to be cast in order for the vote to be seen as legitimate: it wouldn't make sense if in an ecosystem of 10 million possible votes a proposal passed 2 yes votes to 1 no vote. In the Compound ecosystem at least 400,000 COMP are required to vote yes for a proposal to pass. In todays money ($155 per comp) thats over $60 Million dollars worth of votes needed to get something to pass. Needless to say, if you customize this, you will want to pick a number you feel is reasonable for your system. 

**ProposalThreshold (`proposalThreshold()`)** 

To prevent a system where countless spam proposals are created, a proposal threshold requires an address has a certain number of votes before they can make a proposal. In the case of COMP, it's 100,000. Pick a number that works for you. 

**ProposalMaxOperations ( `proposalMaxOperations()`)** 

This is the maximum number of operations that can be executed in a single proposal. Unless you have a good reason, I would probably leave this alone. 

**VotingDelay (`votingDelay()`)**

 ********This is the length of time between which a proposal can be created and it is available to be voted upon. By requiring at least one block to pass, the governance is protected from Flash Loan attacks that might borrow a large number of tokens, propose a vote, and vote on it all in one block. Unless you have a good reason, I would leave this alone. 

**VotingPeriod ( `votingPeriod()`)** 

The length of time for which proposals are available to be voted upon, with time in Ethereum Blocks. Pick what you feel is reasonable for use case. 

### **TimeLock.sol**

Looking at your `Timelock.sol` source code you will see there is first a `SafeMath` library contract at the top of your file. The `Timelock` contract starts at line 171. There you will see the following: 

```jsx
contract Timelock {
    using SafeMath for unit;

   [...list of events omitted for clarity]

    uint public constant GRACE_PERIOD = 14 days;
    uint public constant MINIMUM_DELAY = 2 days;
    uint public constant MAXIMUM_DELAY = 30 days;
```

The constants available to modify are: 

`GRACE_PERIOD`

`MINIMUM_DELAY`

`MAXIMUM_DELAY`

**GRACE_PERIOD -** Once a transaction has been loaded into a timelock for execution, it is required that someone still "press the button" to have it execute and pay the gas required. The `GRACE_PERIOD` is essentially how long between the time at which a transaction becomes available to execute, and when  the proposal had intended the transaction to be executed (the `eta` component on a Proposal- see line 140 on `COMP.sol`). After the `GRACE_PERIOD` and `eta` combined expire, the transaction is considered stale (see line 255 `Timelock.sol`) and is not possible to execute. 

**MINIMUM_DELAY & MAXIMUM_DELAY -** When deploying the `Timelock.sol` contract, one of the constructor arguments is `delay` (see: line 195 `Timlock.sol`). The `MINIMUM_DELAY` & `MAXIMUM_DELAY` serve to set as hardcoded limits on how long the `Timelock` contract needs to wait before executing a transaction. 

In general, I would recommend leaving these set as they are, but again- if you need something different feel free to customize them. Once you're set, run `npx hardhat compile` again. 

## **hardhat.config.js**:

Use this:

```jsx
require("@nomiclabs/hardhat-waffle");
require("@nomiclabs/hardhat-ethers");


// This is a sample Hardhat task. To learn how to create your own go to
// https://hardhat.org/guides/create-task.html
task("accounts", "Prints the list of accounts", async () => {
  const accounts = await ethers.getSigners();

  for (const account of accounts) {
    console.log(account.address);
  }
});

task("Deploy", "Deploys a COMPound style governance system")
  .addParam("token", "The address to receive the initial supply")
  .addParam("timelock", "The timelock administrator")
  .addParam("guardian", "The governor guardian").setAction(async taskArgs => {

    const { deploy } = require("./scripts/Deploy");

    await deploy({
      tokenRecipient: taskArgs.token,
      timeLockAdmin: taskArgs.timelock,
      guardian: taskArgs.guardian
    });

  })


// You need to export an object to set up your config
// Go to https://hardhat.org/config/ to learn more

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
// One url would be someting like this (I changed one letter on it):
// So you need to create one by yourself
// "https://polygon-mainnet.g.alchemy.com/v2/vWbaAY8gHj8IGgcKWmaKQlYHb91QJdHw"
module.exports = {
  networks: {
    hardhat: {
    },
    polygon: {
      url: "https://polygon-mainnet.g.alchemy.com/v2/here_it_goes_a_hash",
      accounts: [
        "0xPRIVATE_KEY_OF_FIRST_ACCOUNT",
        "0xPRIVATE_KEY_OF_SECOND_ACCOUNT",
        "0xPRIVATE_KEY_OF_THIRD_ACCOUNT"
      ]
    }
  },
  solidity: {
    compilers: [
      {
        version: "0.8.0",
      },
      {
        version: "0.8.2",
      },
      {
        version: "0.5.16",
      },
    ]
  }
}
```

To run it using the built in HardHard environment blockchain use: 

`npx hardhat run scripts/Deploy.js`

You should see the following output (Your addresses and token balance might be different)

```markdown

## **Deployment**:

To deploy a contract using HardHat, we need to first write a script. When we first created our project HardHat created a sample script for us, but we're going to start from scratch. To deploy a COMP governance system we need to deploy our contracts in a specific order as some contracts need the address of the others in their constructor functions: 

First, we will deploy COMP, the token. For this our constructor only needs the address of where to send the initial (fixed) token supply.

Create a `Deploy.js` file in your `/scripts` folder, copy/paste the following code: 

```jsx
const hre = require("hardhat");
const ethers = hre.ethers;

async function main() {

    // Compile our Contracts, just in case
    await hre.run('compile');

    // Get a signer from the HardHard environment
    // Learn about signers here: https://docs.ethers.io/v4/api-wallet.html
    const [tokenRecipient, timelockAdmin, guardian] = await ethers.getSigners();

    // This gets the contract from 
    const Token = await hre.ethers.getContractFactory("Comp");
    const token = await Token.deploy(tokenRecipient.address);
    await token.deployed();
    await token.deployTransaction.wait();

    // Deploy Timelock
    const delay = 172800;
    const Timelock = await ethers.getContractFactory("Timelock");
    const timelock = await Timelock.deploy(timelockAdmin.address, delay);
    await timelock.deployed();
    await timelock.deployTransaction.wait();

    // Deploy Governance
    const Gov = await ethers.getContractFactory("GovernorAlpha");
    const gov = await Gov.deploy(timelock.address, token.address, guardian.address);
    await gov.deployed();
    await gov.deployTransaction.wait();

    console.log(`Token deployed to: ${token.address}`);
    console.log(`TimeLock deployed to: ${timelock.address}`);
    console.log(`GovernorAlpha deployed to: ${gov.address}`)

    const initialBalance = await token.balanceOf(tokenRecipient.address);
    console.log(`${initialBalance / 1e18} tokens transfered to ${tokenRecipient.address}`);
}

main()
    .then(() => process.exit(0))
    .catch(error => {
        console.error(error);
        process.exit(1);
    });

```

To run it using the built in HardHard environment blockchain use: 

`npx hardhat run scripts/Deploy.js`


# Deploy to Network:

Now that we have this working, deploying to a real network requires only one thing

1. Private keys to the addresses you will use as Signers. 

**Update the `hardhat.config.js` file**

To deploy to a network using HardHat we need to first give it some information about the network we want to deploy to. In our case, we are deploying to `Rinkeby` and we're going to use Infura as our network connection. (Sign up at [Infura](https://infura.io/) to get your own API key).

We will also need some private keys, which unfortunately even in 2020 (going on 2021), is still the most cumbersome part of the deployment. 

Add a `networks` property to your  `hardhat.config.js` file exports and as a sub-property add your network (in our case `Rinkeby`) along with an array of private keys.

```jsx
module.exports = {
  networks: {
    rinkeby: {
      url: "https://rinkeby.infura.io/v3/<<Your infura Key>>",
      accounts: ["PrivateKey1", "PrivateKey2", "PrivateKey3"]
    }
  },
  solidity: "0.5.16",
};
```

Now to deploy to your network you type: 

```jsx
npx hardhat run scripts/Deploy.js --network rinkeby
```

## **Create a Task**

Finally, we're using HardHat because it helps automate deployments. Let's create a task so we can deploy our governance directly from the command line. 

Inside your `hardhat.config.js` file, we will add the following task, below the already existing sample task (delete the sample task if you like) 

```markdown
task("Deploy", "Deploys a COMPound style governance system", async () => {
..... we will fill this out.

});
```

Lets reuse our code that we wrote in our script, the goal is to make the deployment script modular and export the function so we can import it in our task. 

```markdown
// Delete this as it will cause our task to run twice
// main()
//     .then(() => process.exit(0))
//     .catch(error => {
//         console.error(error);
//         process.exit(1);
//     });

// Add a module.exports 
module.exports = {
        deploy: main
    }
```

Additionally, we're going to want to pass in the address for the initial token holder, guardian, etc.., so that it's not based on `ethers.getSigners()` which comes from our config file (and requires private keys!)  

Lets pass in an object `{tokenRecipient, timeLockAdmin, guardian}` into our `main()` function so that it looks like this: 

```markdown
async function main({tokenRecipient, timeLockAdmin, guardian})
```

Unlike the output from `getSigners()` these variables will not be signers- there won't be private keys attached, so there will be no `.address` property, instead their value is the address. Remove the `.address` from just these three variables when they are passed into the deployment functions. Your new code should look like this: 

```jsx
const hre = require("hardhat");
const ethers = hre.ethers;

async function main({tokenRecipient, timeLockAdmin, guardian}) {

    // Compile our Contracts, just in case
    await hre.run('compile');

    // This gets the contract from 
    const Token = await hre.ethers.getContractFactory("Comp");
    const token = await Token.deploy(tokenRecipient);
    await token.deployed();
    await token.deployTransaction.wait();
    
    // Deploy Timelock
    const delay = 172800;
    const Timelock = await ethers.getContractFactory("Timelock");
    const timelock = await Timelock.deploy(timeLockAdmin, delay);
    await timelock.deployed();
    await timelock.deployTransaction.wait();

    // Deploy Governance
    const Gov = await ethers.getContractFactory("GovernorAlpha");
    const gov = await Gov.deploy(timelock.address, token.address, guardian);
    await gov.deployed();
    await gov.deployTransaction.wait();

    console.log(`Token deployed to: ${token.address}`);
    console.log(`TimeLock deployed to: ${timelock.address}`);
    console.log(`GovernorAlpha deployed to: ${gov.address}`)

    const initialBalance = await token.balanceOf(tokenRecipient);
    console.log(`${initialBalance / 1e18} tokens transfered to ${tokenRecipient}`);
}

module.exports = {
        deploy: main
    }
```

Now we need to add to the HardHat task the ability to get `tokenRecipient`, `timeLockAdmin`, `guardian` from the command line and pass it to our `main` function. To do this, HardHard allows you to collect Params directly from the command line using the `.addParam()` function and access it inside the task. 

Update the task to look like this:

```jsx
task("Deploy", "Deploys a COMPound style governance system")
.addParam("token", "The address to receive the initial supply")
.addParam("timelock", "The timelock administrator")
.addParam("guardian", "The governor guardian").setAction(async taskArgs => {
    
  const { deploy } = require("./scripts/Deploy");

    await deploy({
      tokenRecipient: taskArgs.token,
      timeLockAdmin: taskArgs.timelock,
      guardian: taskArgs.guardian
    });

})
```

Now to deploy you need only one private key in your `module.exports` in the `hardhat.config.js` file, this private key is used to deploy the contracts, but even if it's compromised, your governance system isn't in danger as they key addresses for the governance system are entered on the CLI. It also means you can deploy as many governance systems as you like, quickly and easily, from the CLI. 

To deploy: 

`npx hardhat Deploy --token 0xAddressToReceivetokens --timelock 0xAddressTimeLockAdmin --guardian 0xAddressGovernorAlphaAdmin --network rinkeby`

You can see the final code here: 

[https://github.com/withtally/Tutorial-Deploy-Governance](https://github.com/withtally/Tutorial-Deploy-Governance)


# CrowdChain - A Crowdfunding Dapp

This is a decentralized crowdfunding platform built with **React.js** for the client-side and **Solidity** for the smart contract, deployed on the **Sepolia testnet** using **Thirdweb**.

## Features

- **Create Campaigns**: Users can create crowdfunding campaigns with a funding goal, description, and deadline.
- **Contribute**: Users can contribute funds to campaigns securely through the blockchain.
- **Campaign Management**: Campaign creators can withdraw funds if the goal is met, or refund contributors if not.
- **Blockchain Integration**: All transactions are secure and transparent, using smart contracts on Ethereum's Sepolia testnet.

## Tech Stack

### Frontend
- **React.js**: Frontend framework
- **Thirdweb SDK**: For interacting with the deployed smart contract
- **Tailwind CSS**: For styling

### Smart Contract
- **Solidity**: For the smart contract
- **Thirdweb**: For deploying and managing smart contracts
- **Sepolia Testnet**: Ethereum testnet used for deployment

## Getting Started

### Prerequisites

Make sure you have the following installed on your system:

- **Node.js**: [Download Node.js](https://nodejs.org/)
- **MetaMask**: For interacting with the blockchain
- **Thirdweb SDK**: For contract deployment and interaction

### Installation

1. Clone the repository:

   ```bash
   https://github.com/arnavbansal2764/Crowdfunding-Web3.git
   ```

2. Navigate to the project directory:

   ```bash
   cd crowdfunding-dapp/client
   ```

3. Install the dependencies:

   ```bash
   npm install
   ```

4. Set up environment variables:

   Create a `.env.local` file in the root directory and add the following variables:

   ```bash
   REACT_APP_THIRDWEB_API_KEY=your-thirdweb-api-key
   REACT_APP_CONTRACT_ADDRESS=your-contract-address
   ```

   - `REACT_APP_THIRDWEB_API_KEY`: API key from Thirdweb
   - `REACT_APP_CONTRACT_ADDRESS`: The smart contract address on the Sepolia testnet

### Deploying the Smart Contract

1. Write the Solidity contract. Here’s an example:

   ```solidity
  // SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.9;

contract CrowdFunding {
    struct Campaign {
        address owner;
        string title;
        string description;
        uint256 target;
        uint256 deadline;
        uint256 amountCollected;
        string image;
        address[] donators;
        uint256[] donations;
    }

    mapping(uint256 => Campaign) public campaigns;

    uint256 public numberOfCampaigns = 0;

    function createCampaign(address _owner, string memory _title, string memory _description, uint256 _target, uint256 _deadline, string memory _image) public returns (uint256) {
        Campaign storage campaign = campaigns[numberOfCampaigns];

        require(campaign.deadline < block.timestamp, "The deadline should be a date in the future.");

        campaign.owner = _owner;
        campaign.title = _title;
        campaign.description = _description;
        campaign.target = _target;
        campaign.deadline = _deadline;
        campaign.amountCollected = 0;
        campaign.image = _image;

        numberOfCampaigns++;

        return numberOfCampaigns - 1;
    }

    function donateToCampaign(uint256 _id) public payable {
        uint256 amount = msg.value;

        Campaign storage campaign = campaigns[_id];

        campaign.donators.push(msg.sender);
        campaign.donations.push(amount);

        (bool sent,) = payable(campaign.owner).call{value: amount}("");

        if(sent) {
            campaign.amountCollected = campaign.amountCollected + amount;
        }
    }

    function getDonators(uint256 _id) view public returns (address[] memory, uint256[] memory) {
        return (campaigns[_id].donators, campaigns[_id].donations);
    }

    function getCampaigns() public view returns (Campaign[] memory) {
        Campaign[] memory allCampaigns = new Campaign[](numberOfCampaigns);

        for(uint i = 0; i < numberOfCampaigns; i++) {
            Campaign storage item = campaigns[i];

            allCampaigns[i] = item;
        }

        return allCampaigns;
    }
}
   ```

2. Deploy the smart contract to Sepolia using **Thirdweb**:

   - Go to Thirdweb and create a project.
   - Deploy the smart contract through their platform.

3. After deployment, copy the contract address and paste it into the `.env.local` file.

### Running the Application

1. Start the React application:

   ```bash
   npm run dev
   ```

2. Open your browser and navigate to:

   ```bash
   http://localhost:5173
   ```



## The functions are accessed like this : 

```
  const publishCampaign = async (form) => {
    try {
      const data = await createCampaign({
				args: [
					address, // owner
					form.title, // title
					form.description, // description
					form.target,
					new Date(form.deadline).getTime(), // deadline,
					form.image,
				],
			});

      console.log("contract call success", data)
    } catch (error) {
      console.log("contract call failure", error)
    }
  }

  const getCampaigns = async () => {
    const campaigns = await contract.call('getCampaigns');

    const parsedCampaings = campaigns.map((campaign, i) => ({
      owner: campaign.owner,
      title: campaign.title,
      description: campaign.description,
      target: ethers.utils.formatEther(campaign.target.toString()),
      deadline: campaign.deadline.toNumber(),
      amountCollected: ethers.utils.formatEther(campaign.amountCollected.toString()),
      image: campaign.image,
      pId: i
    }));

    return parsedCampaings;
  }

  const getUserCampaigns = async () => {
    const allCampaigns = await getCampaigns();

    const filteredCampaigns = allCampaigns.filter((campaign) => campaign.owner === address);

    return filteredCampaigns;
  }

  const donate = async (pId, amount) => {
    const data = await contract.call('donateToCampaign', [pId], { value: ethers.utils.parseEther(amount)});

    return data;
  }

  const getDonations = async (pId) => {
    const donations = await contract.call('getDonators', [pId]);
    const numberOfDonations = donations[0].length;

    const parsedDonations = [];

    for(let i = 0; i < numberOfDonations; i++) {
      parsedDonations.push({
        donator: donations[0][i],
        donation: ethers.utils.formatEther(donations[1][i].toString())
      })
    }

    return parsedDonations;
  }
```

## Testing

- Test your smart contract using tools like **Hardhat** or **Truffle**.
- Use **MetaMask** to simulate transactions on the Sepolia testnet.
- Check logs for issues using the browser console and smart contract logs.

## Deployment

Deploy the React application to a platform like **Vercel** or **Netlify**.

### Build for Production

```bash
npm run build
```

This will create an optimized build of the application in the `build/` directory, ready to be deployed.

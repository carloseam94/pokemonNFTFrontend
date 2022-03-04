<!-- Please remove this file from your project -->
<template>
  <div
    class="
      relative
      items-top
      justify-center
      min-h-screen
      bg-gray-100
      sm:items-center sm:pt-0
    "
  >
    <h1 class="text-center pt-8 text-xl">Mint a PokemonNFT</h1>
    <div class="flex pt-8">
      <button
        class="
          mx-auto
          bg-green-400
          p-3
          rounded-md
          hover:bg-green-500
          text-white
          flex
        "
        @click="connectWalletOrMintNFT"
        :disabled="mintingNFT"
      > 
      <span style="align-self: center;">
         {{ mintingNFT ? "Minting NFT" : walletConnected ? "Mint NFT" : "Connect Wallet" }}
      </span> 
        <span class="ml-3" v-show="mintingNFT">
          <circle-spin></circle-spin>
        </span>
      </button>
    </div>
    <div class="flex mt-8">
      <input
        class="mx-auto rounded border-green-600 border p-1 outline-0"
        style="width: 450px"
        v-show="walletConnected"
        :disabled="mintingNFT"
        type="text"
        id="walletAdressInput"
        placeholder="Mint to this address"
      />
    </div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import VueSpinners from 'vue-spinners'
import PokemonNFT from '../utils/PokemonNFT.json';
import Vue from 'vue';
Vue.use(VueSpinners)

export default {
  data() {
    return {
      CONTRACT_ADDRESS: "0x55ad7dB2860291C7271C8b8929d82322Ff0F71f0",
      walletConnected: false,
      mintingNFT: false,
      account: "",
      mintToWalletAdress: ""
    };
  },
  mounted() {
     this.checkIfWalletIsConnected();
  },
  methods: {
    async checkIfWalletIsConnected() {
      /*
       * First make sure we have access to window.ethereum
       */
      const { ethereum } = window;

      if (!ethereum) {
        console.log("Make sure you have metamask!");
        return;
      } else {
        console.log("We have the ethereum object", ethereum);
      }

      /*
       * Check if we're authorized to access the user's wallet
       */
      const accounts = await ethereum.request({ method: "eth_accounts" });

      /*
       * User can have multiple authorized accounts, we grab the first one if its there!
       */
      if (accounts.length !== 0) {
        this.account = accounts[0];
        this.walletConnected = true;
        this.setupEventListener();
        console.log("Found an authorized account:", this.account);
      } else {
        console.log("No authorized account found");
      }
    },
    async connectWallet() {
      try {
        const { ethereum } = window;
  
        if (!ethereum) {
          alert("Get MetaMask!");
          return;
        }
  
        /*
        * Fancy method to request access to account.
        */
        const accounts = await ethereum.request({ method: "eth_requestAccounts" });
  
        /*
        * Boom! This should print out public address once we authorize Metamask.
        */
        console.log("Connected", accounts[0]);
        this.account = accounts[0];
        this.walletConnected = true;
        this.setupEventListener();
      } catch (error) {
        console.log(error);
      }
    },
    connectWalletOrMintNFT() {
      if (this.walletConnected) {
        this.mintNFT();
      } else {
        this.connectWallet();
      }
    },
    async mintNFT() {
      this.mintToWalletAdress = document.getElementById('walletAdressInput').value;
      if(this.mintToWalletAdress == undefined || this,this.mintToWalletAdress == "") {
        alert("Please type the address")
        return
      }
      
      try {
        const { ethereum } = window;
    
        if (ethereum) {
          const provider = new ethers.providers.Web3Provider(ethereum);
          const signer = provider.getSigner();
          const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS, PokemonNFT.abi, signer);
    
          console.log("Going to pop wallet now to pay gas...")
          let nftTxn = await connectedContract.makeAPokemonNFT(this.mintToWalletAdress);
    
          console.log("Mining...please wait.")
          this.mintingNFT = true;
          await nftTxn.wait();
          
          console.log(`Mined, see transaction: https://mumbai.polygonscan.com/address/${this.CONTRACT_ADDRESS}`);
    
        } else {
          console.log("Ethereum object doesn't exist!");
        }
      } catch (error) {
        console.log(error);
        this.mintingNFT = false;
      }
    },
    // Setup our listener.
   async setupEventListener() {
    // Most of this looks the same as our function askContractToMintNft
    try {
      const { ethereum } = window;

      if (ethereum) {
        // Same stuff again
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const connectedContract = new ethers.Contract(this.CONTRACT_ADDRESS, PokemonNFT.abi, signer);

        connectedContract.on("NewNFTMinted", (from, tokenId) => {
          this.mintingNFT = false;
          console.log(from, tokenId.toNumber())
          alert(`NFT minted: see transaction here https://mumbai.polygonscan.com/address/${this.CONTRACT_ADDRESS} or see NFT here https://testnets.opensea.io/assets/mumbai/${this.CONTRACT_ADDRESS}/${tokenId.toNumber()}`)
        });

        console.log("Setup event listener!")

      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
  }
  },
};
</script>

<template>
  <div class="home">
    <div id="terminal"></div>
    <div class="flicker"></div>
    <div class="scanlines"></div>
  </div>
</template>

<script>
import $ from "jquery";
require("jquery.terminal");
import Web3 from "web3";
import Web3Modal, { isMobile } from "web3modal";
import WalletConnectProvider from "@walletconnect/web3-provider";

var __EVAL = (s) => eval(`void (__EVAL = ${__EVAL}); ${s}`);

export default {
  name: "Home",
  data() {
    return {
      scanlines: "",
      tv: "",
      term: "",
      account: "",
      networks: {
        polygon: 137,
        mumbai: 80001,
      },
      infuraId: process.env.VUE_APP_INFURA_ID,
      network: process.env.VUE_APP_NETWORK,
      web3: "",
    };
  },
  methods: {
    exit() {
      const app = this;
      $(".tv").addClass("collapse");
      app.term.disable();
    },
    set_size() {
      // for window height of 170 it should be 2s
      const app = this;
      var height = $(window).height();
      var width = $(window).width();
      var time = (height * 2) / 170;
      if (app.scanlines[0] !== undefined && app.tv[0] !== undefined) {
        app.scanlines[0].style.setProperty("--time", time);
        app.tv[0].style.setProperty("--width", width);
        app.tv[0].style.setProperty("--height", height);
      }
    },
    help() {
      const app = this;
      app.term.echo("\nWelcome to YOMI Quest, following instructions can be used to play the game:\n");
      app.term.echo("[[b;#fff;]connect]: Establish the connection to your wallet");
      app.term.echo("[[b;#fff;]start]: Start the game subscribing yourself, when subscribed you receive 20 life NFTs needed to play");
      app.term.echo("[[b;#fff;]play]: Show your current level, allowing you to get clues to solve the game");
      app.term.echo("[[b;#fff;]solve]: Solve the current level, you will need to put the phrase just after the word 'solve'");
      app.term.echo("\n");
    },
    clear() {
      const app = this;
      app.term.clear();
    },
    async connect() {
      const app = this;
      app.term.echo("Init connection to wallet, please confirm...");
      let providerOptions = {};
      if (app.infuraId !== undefined) {
        providerOptions = {
          walletconnect: {
            package: WalletConnectProvider,
            options: {
              infuraId: app.infuraId,
            },
          },
        };
      }
      // Instantiating Web3Modal
      const web3Modal = new Web3Modal({
        cacheProvider: true,
        providerOptions: providerOptions,
      });
      const provider = await web3Modal.connect();
      app.web3 = await new Web3(provider);
      // Checking if networkId matches
      const netId = await app.web3.eth.net.getId();
      if (parseInt(netId) !== app.networks[app.network]) {
        app.term.echo("Switch to " + app.network + " network!");
      } else {
        const accounts = await app.web3.eth.getAccounts();
        if (accounts.length > 0) {
          app.account = accounts[0];
          app.term.echo("Connected to account: " + app.account);
        }
      }
    },
    async start() {
      const app = this;
      app.term.echo("Please confirm action in your wallet...");
      
    },
  },
  mounted() {
    const app = this;
    app.scanlines = $(".scanlines");
    app.tv = $(".tv");
    app.term = $("#terminal").terminal(
      function (command, term) {
        const cmd = $.terminal.parse_command(command);
        if (cmd.command !== undefined) {
          try {
            app[cmd.command]();
          } catch (e) {
            term.error(new String(e));
          }
        }
      },
      {
        name: "yomi-quest",
        greetings:
          `Welcome to YOMI Quest, an on-chain game based on cryptographic proofs.\n`,
        onResize: app.set_size(),
        exit: false,
        // detect iframe codepen preview
        enabled: $("body").attr("onload") === undefined,
        onInit: function () {
          app.set_size();
          this.echo(
            "Type and execute [[b;#fff;]help] function to get instructions on how to proceed.\nNo worries, this is just a game, you're no hacking and your computer is completely safe!\n\n"
          );
        },
        prompt: "web3> ",
      }
    );
  },
};
</script>

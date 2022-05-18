<template>
  <div class="home">
    <div
      id="audio"
      style="
        position: fixed;
        width: 100%;
        bacgkround: red;
        height: 90px;
        bottom: 0;
        left: 0;
        z-index: -1;
        opacity: -1;
      "
    ></div>
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
const keccak256 = require("keccak256");
const { MerkleTree } = require("merkletreejs");
const axios = require("axios");
var __EVAL = (s) => eval(`void (__EVAL = ${__EVAL}); ${s}`);
export default {
  name: "Home",
  data() {
    return {
      scanlines: "",
      tv: "",
      term: "",
      account: "",
      axios: axios,
      networks: {
        polygon: 137,
        mumbai: 80001,
        rinkeby: 4,
        mainnet: 1,
      },
      infuraId: process.env.VUE_APP_INFURA_ID,
      network: process.env.VUE_APP_NETWORK,
      contract: process.env.VUE_APP_CONTRACT_ADDRESS,
      web3: "",
      ABI: [
        {
          inputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          name: "nft_kind",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "_owner",
              type: "address",
            },
          ],
          name: "tokensOfOwner",
          outputs: [
            {
              internalType: "uint256[]",
              name: "ownerTokens",
              type: "uint256[]",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          name: "game_prices",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          name: "levels",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          name: "game_instructions",
          outputs: [
            {
              internalType: "string",
              name: "",
              type: "string",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [],
          name: "subscribe",
          outputs: [],
          stateMutability: "payable",
          type: "function",
        },
        {
          inputs: [],
          name: "BIRTH_FEE",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "address",
              name: "",
              type: "address",
            },
          ],
          name: "birthdays",
          outputs: [
            {
              internalType: "uint256",
              name: "",
              type: "uint256",
            },
          ],
          stateMutability: "view",
          type: "function",
        },
        {
          inputs: [
            {
              internalType: "uint256",
              name: "_game",
              type: "uint256",
            },
            {
              internalType: "bytes32[]",
              name: "_merkleProof",
              type: "bytes32[]",
            },
            {
              internalType: "string",
              name: "_solution",
              type: "string",
            },
            {
              internalType: "uint256",
              name: "_tokenId",
              type: "uint256",
            },
          ],
          name: "solveGame",
          outputs: [],
          stateMutability: "payable",
          type: "function",
        },
      ],
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
      app.term.echo(
        "\nWelcome to YOMI Quest, following instructions can be used to play the game:\n"
      );
      app.term.echo(
        "[[b;#fff;]connect]: Establish the connection to your wallet"
      );
      app.term.echo(
        "[[b;#fff;]register]: Subscribe to game, when subscribed you receive 20 life NFTs needed to play, this can be done once per address"
      );
      app.term.echo(
        "[[b;#fff;]play]: Show your current level, allowing you to get clues to solve the game"
      );
      app.term.echo(
        "[[b;#fff;]solve]: Solve the current level, you will need to put the phrase just after the word 'solve'"
      );
      app.term.echo(
        "[[b;#fff;]love]: Follow YOMI on Twitter and spread the word!"
      );
      app.term.echo("\n");
    },
    love() {
      window.open("https://twitter.com/YOMI_WEB3", "_blank");
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
          $("#audio").html(
            '<audio id="player" controls><source src="/sounds/rocket.mp3"></audio>'
          );
          const playInterval = setInterval(function () {
            try {
              $("#player").trigger("play");
              clearTimeout(playInterval);
            } catch (e) {
              console.log(e);
              console.log("CAN'T PLAY");
            }
          }, 200);
        }
      }
    },
    async register() {
      const app = this;
      if (app.account.length > 0) {
        app.term.echo("Checking if you born yet...");
        const contract = new app.web3.eth.Contract(app.ABI, app.contract);
        const birthday = await contract.methods.birthdays(app.account).call();
        if (parseInt(birthday) === 0) {
          app.term.echo("Please confirm operation in your wallet...");
          const birth_fee = await contract.methods.BIRTH_FEE().call();
          console.log("Birth fee is:", birth_fee);
          const tx = await contract.methods
            .subscribe()
            .send({
              from: app.account,
              value: birth_fee.toString(),
            })
            .on("transactionHash", (pending) => {
              app.term.echo("Waiting for transaction at " + pending + "...");
            });
          app.term.echo("Successfully subscribed!");
          $("#audio").html(
            '<audio id="player" controls><source src="/sounds/coin.mp3"></audio>'
          );
          const playInterval = setInterval(function () {
            try {
              $("#player").trigger("play");
              clearTimeout(playInterval);
            } catch (e) {
              console.log(e);
              console.log("CAN'T PLAY");
            }
          }, 200);
          console.log(tx);
        } else {
          app.term.echo(
            "You already subscribed, please write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
          );
        }
      } else {
        app.term.echo(
          "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
        );
      }
    },
    async play() {
      const app = this;
      if (app.account.length > 0) {
        const contract = new app.web3.eth.Contract(app.ABI, app.contract);
        const birthday = await contract.methods.birthdays(app.account).call();
        if (parseInt(birthday) > 0) {
          app.term.echo("Checking your level...");
          const level = await contract.methods.levels(app.account).call();
          app.term.echo("Your level is: " + level);
          app.term.echo("Fetching instructions from IPFS...");
          const game_instructions = await contract.methods
            .game_instructions(level)
            .call();
          if (game_instructions !== undefined && game_instructions.length > 0) {
            $("#audio").html(
              '<audio id="player" controls loop><source src="/sounds/play.mp3"></audio>'
            );
            const playInterval = setInterval(function () {
              try {
                $("#player").trigger("play");
                clearTimeout(playInterval);
              } catch (e) {
                console.log(e);
                console.log("CAN'T PLAY");
              }
            }, 200);
            app.term.echo("Found instructions endpoint: " + game_instructions);
            app.term.echo("Downloading instructions, please wait..");
            const instructions = await axios.get(
              game_instructions.replace(
                "ipfs://",
                "https://ipfs.yomi.digital/ipfs/"
              )
            );
            if (instructions.data.image !== undefined) {
              var img = new Image();
              img.onload = function () {
                var canvas = $("<canvas/>");
                var context = canvas[0].getContext("2d");
                canvas.attr({
                  width: img.width,
                  height: img.height,
                });
                app.term.echo(function () {
                  var cols = app.term.cols();
                  var width = cols / 2;
                  var height = (width * img.height) / img.width;
                  // height / 2 because dimension of single character is not square
                  context.drawImage(img, 0, 0, width, height / 2);
                  try {
                    app.term.echo(
                      app.asciify(context, " `~:*iVOEM", width, height / 2)
                    );
                    app.term.echo(
                      "\n[[b;#fff;]" + instructions.data.name + "]"
                    );
                    app.term.echo("\n" + instructions.data.description);
                    app.term.echo(
                      "\n\n--\n\nNow it's time to solve, when you're ready type:"
                    );
                    app.term.echo(
                      "\n[[b;#fff;]solve] `YOUR SOLUTION` and press [[b;#fff;]ENTER]"
                    );
                    return "\n";
                  } catch (e) {
                    app.term.exception(e);
                    return "";
                  }
                });
              };
              axios
                .get(
                  instructions.data.image.replace(
                    "ipfs://",
                    "https://ipfs.yomi.digital/ipfs/"
                  ),
                  { responseType: "arraybuffer" }
                )
                .then((response) => {
                  let blob = new Blob([response.data], {
                    type: response.headers["content-type"],
                  });
                  img.src = URL.createObjectURL(blob);
                });
            } else {
              app.term.echo("\n[[b;#fff;]" + instructions.data.name + "]");
              app.term.echo("\n" + instructions.data.description);
              app.term.echo(
                "\n\nNow it's time to solve, when you're ready type:"
              );
              app.term.echo(
                "\n\n--\n\n[[b;#fff;]solve] `YOUR SOLUTION` and press [[b;#fff;]ENTER]"
              );
              app.term.echo("\n");
            }
          } else {
            $("#audio").html(
              '<audio id="player" controls><source src="/sounds/completed.mp3"></audio>'
            );
            const playInterval = setInterval(function () {
              try {
                $("#player").trigger("play");
                clearTimeout(playInterval);
              } catch (e) {
                console.log(e);
                console.log("CAN'T PLAY");
              }
            }, 200);
            app.term.echo("");
            app.term.echo("");
            app.term.echo(
              "Thanks for completing this beta version of YOMI QUEST!"
            );
            app.term.echo(
              "If you liked this project please write [[b;#fff;]love] and press [[b;#fff;]ENTER]"
            );
            app.term.echo("");
            app.term.echo("");
          }
        } else {
          app.term.echo(
            "Need to setup your account first, please write [[b;#fff;]register] and press [[b;#fff;]ENTER]!"
          );
        }
      } else {
        app.term.echo(
          "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
        );
      }
    },
    async solve(quest) {
      const app = this;
      let levelBefore;
      if (app.account.length > 0) {
        if (quest.length > 0) {
          app.clear();
          const contract = new app.web3.eth.Contract(app.ABI, app.contract);
          const birthday = await contract.methods.birthdays(app.account).call();
          if (parseInt(birthday) > 0) {
            app.term.echo("Checking your level...");
            const level = await contract.methods.levels(app.account).call();
            levelBefore = level;
            app.term.echo("Your level is: " + level);

            quest = quest.map((w) => w.toUpperCase());
            const solution = quest[Math.floor(Math.random() * quest.length)];
            const price = await contract.methods.game_prices(level).call();
            app.term.echo(
              "Trying to solve paying " +
                app.web3.utils.fromWei(price, "ether") +
                " ETH"
            );

            let leaves = await quest.map((x) => keccak256(x));
            let tree = await new MerkleTree(leaves, keccak256, {
              sortPairs: true,
            });
            const proof = tree.getHexProof(keccak256(solution));

            app.term.echo("Cryptographic proof is: " + JSON.stringify(proof));
            const tokensOfOwnerBefore = await contract.methods
              .tokensOfOwner(app.account)
              .call();
            app.term.echo(
              "You own " +
                tokensOfOwnerBefore.length +
                " NFTs, searching useful token for coin."
            );
            if (tokensOfOwnerBefore.length > 0) {
              let coin;
              let found = false;
              let r = 0;
              while (found === false) {
                const kind = await contract.methods
                  .nft_kind(tokensOfOwnerBefore[r])
                  .call();
                if (parseInt(kind) === 0) {
                  found = true;
                  coin = tokensOfOwnerBefore[r];
                  app.term.echo("Found token to use as coin: " + coin);
                }
                r++;
                if (r === tokensOfOwnerBefore.length) {
                  found = true;
                }
              }
              if (coin !== undefined) {
                try {
                  await contract.methods
                    .solveGame(level, proof, solution, coin)
                    .send({
                      from: app.account,
                      value: price.toString(),
                    })
                    .on("transactionHash", (pending) => {
                      app.term.echo(
                        "Waiting for transaction at " + pending + "..."
                      );
                    });
                  const levelAfter = await contract.methods
                    .levels(app.account)
                    .call();
                  app.term.echo("\n--\nTransaction completed!");
                  app.term.echo("Your level now is: " + levelAfter);
                  if (levelBefore === levelAfter) {
                    app.term.echo("\n--\nOh NO!\n\nAnswer was wrong...retry!!");
                    $("#audio").html(
                      '<audio id="player" controls><source src="/sounds/wrong.mp3"></audio>'
                    );
                    const playInterval = setInterval(function () {
                      try {
                        $("#player").trigger("play");
                        clearTimeout(playInterval);
                      } catch (e) {
                        console.log(e);
                        console.log("CAN'T PLAY");
                      }
                    }, 200);
                  } else {
                    app.term.echo(
                      "\n--\nOh YEAH!\n\nAnswer was correct, go ahead!!"
                    );
                    $("#audio").html(
                      '<audio id="player" controls><source src="/sounds/clap.mp3"></audio>'
                    );
                    const playInterval = setInterval(function () {
                      try {
                        $("#player").trigger("play");
                        clearTimeout(playInterval);
                      } catch (e) {
                        console.log(e);
                        console.log("CAN'T PLAY");
                      }
                    }, 200);
                  }
                  const tokensOfOwnerAfter = await contract.methods
                    .tokensOfOwner(app.account)
                    .call();
                  app.term.echo(
                    "You own " + tokensOfOwnerAfter.length + " NFTs now!"
                  );
                } catch (e) {
                  app.term.echo("Oh no! There's an error!");
                  app.term.echo(e.message);
                }
              } else {
                app.term.echo("Can't find coins on your wallet!");
              }
            } else {
              app.term.echo("Sorry, you must own at least 1 NFT to play!");
            }
          } else {
            app.term.echo(
              "Need to setup your account first, please write [[b;#fff;]register] and press [[b;#fff;]ENTER]!"
            );
          }
        } else {
          app.term.echo("Need to write a solution after [[b;#fff;]solve]!");
        }
      } else {
        app.term.echo(
          "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
        );
      }
    },
    color(r, g, b) {
      for (var i = 0; i < arguments.length; ++i) {
        arguments[i] = arguments[i].toString(16);
        if (arguments[i].length == 1) {
          arguments[i] = "0" + arguments[i];
        }
      }
      return [].join.call(arguments, "");
    },
    asciify(ctx, palette, width, height) {
      const app = this;
      var ascii = "";
      var pixels = ctx.getImageData(0, 0, width, height);
      for (var y = 0; y < height - 1; ++y) {
        for (var x = 0; x < width - 1; ++x) {
          var p = 4 * (x + pixels.width * y);
          var r = pixels.data[p++];
          var g = pixels.data[p++];
          var b = pixels.data[p++];
          var v = Math.max(r, g, b) / 255;
          //v = 1 - Math.pow(v, GAMMA);
          v = (v * palette.length) >> 0;
          v = Math.max(0, Math.min(palette.length - 1, v));
          ascii += "[[;#" + app.color(r, g, b) + ";]" + palette[v] + "]";
        }
        if (y < height - 1) ascii += "\n";
      }
      return ascii;
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
            app[cmd.name](cmd.args);
          } catch (e) {
            term.error(new String(e));
          }
        }
      },
      {
        name: "yomi-quest",
        greetings: ``,
        onResize: app.set_size(),
        scrollOnEcho: true,
        exit: false,
        // detect iframe codepen preview
        enabled: $("body").attr("onload") === undefined,
        onInit: function () {
          app.set_size();
          $("#audio").html(
            '<audio id="player" controls loop><source src="/sounds/enter.mp3"></audio>'
          );
          const playInterval = setInterval(function () {
            try {
              $("#player").trigger("play");
              clearTimeout(playInterval);
            } catch (e) {
              console.log(e);
              console.log("CAN'T PLAY");
            }
          }, 200);
          var img = new Image();
          img.onload = function () {
            var canvas = $("<canvas/>");
            var context = canvas[0].getContext("2d");
            canvas.attr({
              width: img.width,
              height: img.height,
            });
            app.term.echo(function () {
              var cols = app.term.cols();
              var width = cols / 5;
              var height = (width * img.height) / img.width;
              // height / 2 because dimension of single character is not square
              context.drawImage(img, 0, 0, width, height / 2);
              try {
                app.term.echo(
                  app.asciify(context, " `~:*iVOEM", width, height / 2)
                );
                app.term.echo(
                  "Welcome to YOMI Quest Demo,\nan on-chain game based on cryptographic proofs.\n\nWe're playing on Rinkeby network, so right now you'll not spend nor gain real money.\nYou will try to resolve 5 different quests, do your best to be the first!\n\nA public list of champions will be available as soon as the game start and\nyou'll be able to win real NFTs from our Autonomous Design collection.\n\n\n--"
                );
                app.term.echo(
                  "\n\nType [[b;#fff;]help] and press [[b;#fff;]ENTER] to get instructions on how to proceed.\nNo worries, this is just a game, you're no hacking and your computer is completely safe!\n\nTurn ON audio for a better experience!!"
                );
                return "\n";
              } catch (e) {
                app.term.exception(e);
                return "";
              }
            });
          };
          axios
            .get(
              "https://ipfs.yomi.digital/ipfs/QmcYHMf9mGzmhZgyRuYpzkM9cmYJ1c3J5VWyywUW3HDnEF",
              { responseType: "arraybuffer" }
            )
            .then((response) => {
              let blob = new Blob([response.data], {
                type: response.headers["content-type"],
              });
              img.src = URL.createObjectURL(blob);
            });
        },
        prompt: "web3> ",
      }
    );
  },
};
</script>

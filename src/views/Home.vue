<template>
  <div class="home">
    <div id="audio" class="sounds"></div>
    <div id="base" class="sounds"></div>
    <div id="terminal"></div>
    <div class="flicker"></div>
    <div class="scanlines"></div>
    <div
      id="died"
      v-if="died"
      style="
        position: fixed;
        top: 0;
        left: 0;
        width: 100vw;
        height: 100vh;
        background: #000;
      "
    >
      <video
        style="width: 100vw; height: 100vh"
        width="100%"
        height="100%"
        id="died_video"
        controls
        loop
      >
        <source src="/you_died.mp4" type="video/mp4" />
        <source src="/you_died.ogv" type="video/ogg" />
        <source src="/you_died.webm" type="video/webm" />
      </video>
      <div
        style="
          position: absolute;
          top: 0;
          left: 0;
          z-index: 999;
          font-size: 8rem;
          width: 100%;
          height: 100vh;
          line-height: calc(100vh / 2);
          color: #fff;
          background: rgba(0, 0, 0, 0.2);
          text-align: center;
          font-weight: bold;
        "
        v-html="diedText"
      ></div>
    </div>
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
const abi = require("../abi.json");

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
      apiUrl: process.env.VUE_APP_API_URL,
      web3: "",
      ABI: abi,
      isWorking: false,
      died: false,
      diedText: "",
      timer: "",
      prompt: "",
      spinner: {
        interval: 80,
        frames: ["⣾", "⣽", "⣻", "⢿", "⡿", "⣟", "⣯", "⣷"],
      },
    };
  },
  methods: {
    wait() {
      const app = this;
      app.isWorking = true;
      app.prompt = app.term.get_prompt();
      let i = 0;
      function set() {
        const text = app.spinner.frames[i++ % app.spinner.frames.length];
        app.term.set_prompt(text);
      }
      app.term.find(".cursor").hide();
      set();
      app.timer = setInterval(set, app.spinner.interval);
    },
    done() {
      const app = this;
      clearInterval(app.timer);
      setTimeout(function () {
        app.isWorking = false;
        app.term.set_prompt(app.prompt);
        app.term.find(".cursor").show();
      }, app.spinner.interval);
    },
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
      app.term.echo("[[b;#fff;]stats]: Show some stats about the game!");
      app.term.echo(
        "[[b;#fff;]vault]: Check your balance inside contract's vault."
      );
      app.term.echo(
        "[[b;#fff;]withdraw]: Withdraw all the funds from contract's vault."
      );
      app.term.echo(
        "[[b;#fff;]love]: Follow YOMI on Twitter and spread the word!"
      );
      app.term.echo(
        "[[b;#fff;]sos]: Enter YOMIVERSE and get help from the community."
      );
      app.term.echo(
        "[[b;#fff;]contribute]: YOMI Quest is open-source!"
      );
      app.term.echo("\n");
    },
    love() {
      window.open("https://twitter.com/YOMI_WEB3", "_blank");
    },
    contribute() {
      window.open("https://github.com/yomi-digital/quest-ui", "_blank");
    },
    sos() {
      window.open("https://discord.gg/2EaXhk9zfE", "_blank");
    },
    clear() {
      const app = this;
      app.term.clear();
    },
    async connect() {
      const app = this;
      if (!app.isWorking) {
        app.wait();
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
          app.term.echo(
            "Switching to " +
              app.network +
              " network, please confirm on your wallet.."
          );
          try {
            await window.ethereum.request({
              method: "wallet_switchEthereumChain",
              params: [
                {
                  chainId: "0x4",
                },
              ],
            });
            app.done();
            setTimeout(function () {
              app.connect();
            }, 100);
          } catch (e) {
            app.term.echo(
              "Can't automatically switch to rinkeby, please do it manually."
            );
          }
        } else {
          const accounts = await app.web3.eth.getAccounts();
          if (accounts.length > 0) {
            app.account = accounts[0];
            app.term.echo("Connected to account: " + app.account);
            app.term.echo("Checking if some spare ETH on Rinkeby are needed..");
            const faucet = await axios.get(
              app.apiUrl + "/faucet/" + app.account
            );
            if (faucet.data.indexOf("0x") !== -1) {
              app.term.echo(
                "Just received a small tip from faucet at " + faucet.data + "!"
              );
            } else {
              app.term.echo("You have enough balance to play..proceed!");
            }
            console.log("FAUCET_RESPONSE", faucet.data);
            $("#audio").html(
              '<audio id="player" controls><source src="/sounds/rocket.mp3"></audio>'
            );
            const contract = new app.web3.eth.Contract(app.ABI, app.contract);
            const birthday = await contract.methods
              .birthdays(app.account)
              .call();
            if (parseInt(birthday) === 0) {
              app.term.echo(
                "You need to subscribe now, please write [[b;#fff;]register] and press [[b;#fff;]ENTER]!"
              );
            } else {
              app.term.echo(
                "You already subscribed, please write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
              );
            }
            app.done();
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
            app.done();
          }
        }
      }
    },
    async register() {
      const app = this;
      if (!app.isWorking) {
        if (app.account.length > 0) {
          app.wait();
          app.term.echo("Checking if you born yet...");
          const contract = new app.web3.eth.Contract(app.ABI, app.contract);
          const birthday = await contract.methods.birthdays(app.account).call();
          if (parseInt(birthday) === 0) {
            app.term.echo("Please confirm operation in your wallet...");
            const birth_fee = await contract.methods.BIRTH_FEE().call();
            console.log("Birth fee is:", birth_fee);
            try {
              await contract.methods
                .subscribe()
                .send({
                  from: app.account,
                  value: birth_fee.toString(),
                })
                .on("transactionHash", (pending) => {
                  app.term.echo(
                    "Waiting for transaction at " + pending + "..."
                  );
                });
              app.term.echo("Successfully subscribed!");
              $("#audio").html(
                '<audio id="player" controls><source src="/sounds/coin.mp3"></audio>'
              );
              app.term.echo(
                "Now it's time to start, please write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
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
              app.done();
            } catch (e) {
              app.term.echo(e.message);
              app.done();
            }
          } else {
            app.term.echo(
              "You already subscribed, please write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
            );
            app.done();
          }
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
      }
    },
    async play() {
      const app = this;
      if (!app.isWorking) {
        if (app.account.length > 0) {
          app.wait();
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
            if (
              game_instructions !== undefined &&
              game_instructions.length > 0
            ) {
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
              app.term.echo(
                "Found instructions endpoint: " + game_instructions
              );
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
                      app.done();
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
                app.done();
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
              app.done();
            }
          } else {
            app.done();
            app.term.echo(
              "Need to setup your account first, please write [[b;#fff;]register] and press [[b;#fff;]ENTER]!"
            );
          }
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
      }
    },
    async solve(quest) {
      const app = this;
      if (!app.isWorking) {
        let levelBefore;
        if (app.account.length > 0) {
          if (quest.length > 0) {
            app.wait();
            const contract = new app.web3.eth.Contract(app.ABI, app.contract);
            const birthday = await contract.methods
              .birthdays(app.account)
              .call();
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
                      app.term.echo(
                        "\n--\nOh NO!\n\nAnswer was wrong...retry!!"
                      );
                      $("#audio").html(
                        '<audio id="player" controls><source src="/sounds/wrong.mp3"></audio>'
                      );
                      app.term.echo(
                        "You need to show again the game? Write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
                      );
                      app.done();
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
                      app.term.echo(
                        "It's time to solve another quest, write [[b;#fff;]play] and press [[b;#fff;]ENTER]!"
                      );
                      $("#audio").html(
                        '<audio id="player" controls><source src="/sounds/clap.mp3"></audio>'
                      );
                      app.done();
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
                  } catch (e) {
                    app.term.echo("Oh no! There's an error!");
                    app.term.echo(e.message);
                    app.done();
                  }
                } else {
                  app.term.echo("Can't find coins on your wallet!");
                  app.done();
                }
              } else {
                app.term.echo("Sorry, you died!");
                app.died = true;
                $("#audio").html("");
                setTimeout(function () {
                  $("#died_video").trigger("play");
                });
                setInterval(function () {
                  if (app.diedText.length === 0) {
                    app.diedText = "YOU<br>DIED";
                  } else {
                    app.diedText = "";
                  }
                }, 300);
                app.done();
              }
            } else {
              app.term.echo(
                "Need to setup your account first, please write [[b;#fff;]register] and press [[b;#fff;]ENTER]!"
              );
              app.done();
            }
          } else {
            app.term.echo("Need to write a solution after [[b;#fff;]solve]!");
          }
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
      }
    },
    async stats() {
      const app = this;
      if (!app.isWorking) {
        if (app.account.length > 0) {
          app.wait();
          const contract = new app.web3.eth.Contract(app.ABI, app.contract);
          app.term.echo("List of champions:");
          for (let i = 0; i <= 4; i++) {
            let winner = await contract.methods.winners(i).call();
            if (winner !== "0x0000000000000000000000000000000000000000") {
              try {
                const ens = await axios.get(app.apiUrl + "/ens/" + winner);
                if (ens.data !== "NOT_FOUND") {
                  winner += " (" + ens.data + ")";
                }
              } catch (e) {
                console.log(e.message);
              }
            }
            app.term.echo("[[b;#fff;]Level " + (i + 1) + "]: " + winner);
          }
          app.term.echo("\n\nList of retries for each level:");
          for (let i = 0; i <= 4; i++) {
            const losers = await contract.methods.losers(i).call();
            app.term.echo("[[b;#fff;]Level " + (i + 1) + "]: " + losers);
          }
          app.done();
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
      }
    },
    async vault() {
      const app = this;
      if (!app.isWorking) {
        if (app.account.length > 0) {
          app.wait();
          const contract = new app.web3.eth.Contract(app.ABI, app.contract);
          const vault = await contract.methods.vaults(app.account).call();
          const vault_eth = parseFloat(app.web3.utils.fromWei(vault, "ether"));
          app.term.echo("Your vault is: " + vault_eth + " ETH");
          if (vault_eth > 0) {
            app.term.echo(
              "You've some royalties to take, please write [[b;#fff;]withdraw] and press [[b;#fff;]ENTER]!"
            );
          }
          app.done();
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
      }
    },
    async withdraw() {
      const app = this;
      if (!app.isWorking) {
        if (app.account.length > 0) {
          app.wait();
          const contract = new app.web3.eth.Contract(app.ABI, app.contract);
          const vault = await contract.methods.vaults(app.account).call();
          const vault_eth = parseFloat(app.web3.utils.fromWei(vault, "ether"));
          if (vault_eth > 0) {
            app.term.echo(
              "Withdrawing " +
                vault_eth +
                " ETH from contract, confirm on wallet.."
            );
            try {
              await contract.methods
                .withdrawFunds()
                .send({
                  from: app.account,
                })
                .on("transactionHash", (pending) => {
                  app.term.echo(
                    "Waiting for transaction at " + pending + "..."
                  );
                });
              app.term.echo("Withdraw was successful!");
              app.done();
              $("#audio").html(
                '<audio id="player" controls><source src="/sounds/coin.mp3"></audio>'
              );
            } catch (e) {
              app.term.echo(e.message);
              app.done();
            }
          } else {
            app.term.echo("Nothing to withdraw fren!");
            app.done();
          }
        } else {
          app.term.echo(
            "Need to connect first, please write [[b;#fff;]connect] and press [[b;#fff;]ENTER]!"
          );
        }
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
            if (
              app[cmd.name] !== undefined &&
              cmd.name !== "done" &&
              cmd.name !== "wait"
            ) {
              app.clear();
              app[cmd.name](cmd.args);
            } else {
              app.term.echo("Command not recognized, please retry!");
            }
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
          $("#base").html(
            '<audio id="background" controls loop><source src="/sounds/enter.mp3"></audio>'
          );
          setInterval(function () {
            try {
              $("#background").trigger("play");
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
              var width = 35;
              var height = 30;
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

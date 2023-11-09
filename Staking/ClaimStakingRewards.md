# Claim Staking Rewards

## **How to check the reward amount**

Unfortunately, the amount of accumulated rewards are not directly stored on the chain but divided into multiple storage entries. Luckily, you can easily query your current reward status by performing a runtime API call which we created for that specific purpose. Since this is just a simple query, it does not cost any transaction fees.

### Polkadot Apps

In the Polkadot JS Apps go to `Developer -> Runtime calls`.

1. Select the `parachainStaking` endpoint.
2. Select the `getUnclaimedStakingRewards(account)` call.
3. Select your Curio address (the *account: AccountId32* field)
4. Submit the runtime call (the *Submit Runtime call* button). You do not need to sign or pay any fees.

## How to claim rewards for collators

You can do it with the script. Firstly, you should install rust using the steps described in the `Install required packages and Rust` block (if you already have rust installed on your computer, then you can proceed to the second step `Install and run script`).

### Install required packages and Rust

To install the Rust toolchain on Linux:

1. Log on to your computer and open a terminal shell.
2. Check the packages you have installed on the local computer by running an appropriate package management command for your Linux distribution.
3. Add any package dependencies you are missing to your local development environment by running an appropriate package management command for your Linux distribution.
    
    For example, on Ubuntu Desktop or Ubuntu Server, you might run a command similar to the following:
    
    ```bash
    sudo apt install --assume-yes git clang curl libssl-dev protobuf-compiler
    ```
    
    Click the tab titles to see examples for other Linux operating systems:
    
    - Debian:
    
    ```bash
    sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
    ```
    
    - Arch
    
    ```bash
    pacman -Syu --needed --noconfirm curl git clang make protobuf
    ```
    
    - fedora
    
    ```bash
    sudo dnf update
    sudo dnf install clang curl git openssl-devel make protobuf-compiler
    ```
    
    - opensuse
    
    ```bash
    sudo zypper install clang curl git openssl-devel llvm-devel libudev-devel make protobuf
    ```
    
    Remember that different distributions might use different package managers and bundle packages in different ways.
    For example, depending on your installation selections, Ubuntu Desktop and Ubuntu Server might have different packages and different requirements.
    However, the packages listed in the command-line examples are applicable for many common Linux distributions, including Debian, Linux Mint, MX Linux, and Elementary OS.
    
4. Download the `rustup` installation program and use it to install Rust by running the following command:
    
    ```bash
    curl --proto '=https' --tlsv1.2 -sSf <https://sh.rustup.rs> | sh
    ```
    
5. Follow the prompts displayed to proceed with a default installation.
6. Update your current shell to include Cargo by running the following command:
    
    ```bash
    source $HOME/.cargo/env
    ```
    
7. Verify your installation by running the following command:
    
    ```bash
    rustc --version
    ```
    
8. Configure the Rust toolchain to default to the latest stable version by running the following commands:
    
    ```bash
    rustup default stable
    rustup update
    ```
    
9. Add the `nightly` release and the `nightly` WebAssembly (wasm) targets to your development environment by running the following commands:
    
    ```bash
    rustup update nightly
    rustup target add wasm32-unknown-unknown --toolchain nightly
    ```
    
10. Verify the configuration of your development environment by running the following command:
    
    ```bash
    rustup show
    rustup +nightly show
    ```
    
    The command displays output similar to the following:
    
    ```bash
    # rustup show
    
    active toolchain
    ----------------
    
    stable-x86_64-unknown-linux-gnu (default)
    rustc 1.62.1 (e092d0b6b 2022-07-16)
    
    # rustup +nightly show
    
    active toolchain
    ----------------
    
    nightly-x86_64-unknown-linux-gnu (overridden by +toolchain on the command line)
    rustc 1.65.0-nightly (34a6cae28 2022-08-09)
    ```
    
    ### Install and run script
    
    To install script:
    
    1. Log on to your computer and open a terminal shell.
    2. Clone the curio-parachain-staking-script by running the following command:
    
    ```bash
    git clone https://gitlab.com/curioteam/curio-parachain-staking-script.git
    ```
    
    1. Change to the root of the curio-parachain-staking-script directory by running the following command:
    
    ```bash
    cd curio-parachain-staking-script
    ```
    
    1. Compile the curio-parachain-staking-script by running the following command:
    
    ```bash
    cargo build
    ```
    
    To run script:
    
    1. Create a .env file in the root of curio-parachain-staking-script directory by running the following command (for example using the `nano` editor):
    
    ```bash
    nano .env
    ```
    
    1. In the created and opened file, insert the following information (the data after equals is given for example):
    
    ```bash
    WS_NODE_URL=ws://localhost:9944
    ACCOUNT_KEY=31060e858b5326a87fed3c05b73ddea939d343996cb4cfd85ec94537f83377df
    CLAIMING_TYPE=1_SESSIONS
    ```
    
    - `WS_NODE_URL` - your url where you started the node;
    - `ACCOUNT_KEY` - your secret seed that you have been generated with subkey, for example, without 0x;
    - `CLAIMING_TYPE` - the period for claiming rewards in the network (format: `N_PERIOD`; `N` - digit (more than 0), `PERIOD` - one of: SESSIONS or BLOCKS).
    
    You can also see the `.env.example` file as an example by running the following command:
    
    ```bash
    nano .env.example
    ```
    
    1. Press `Ctrl+S` and then `Ctrl+X` to save the file and exit the `nano` editor sequential.
    2. Now you can run the script by running the following command:
    
    ```bash
    cargo run
    ```
    
    ### Rerun
    
    If you want to change something in config, you should:
    
    1. Press `Ctrl+C` to stop a running script from the terminal window in which you ran it.
    2. Open `.env` file and change what you want.
    3. Then you need to rebuild script by running the following command:
    
    ```bash
    cargo build
    ```
    
    1. And you should start your script by running the following command:
    
    ```bash
    cargo run
    ```
    
    ### Provided output to the command line
    
    In order to understand whether the script is working, it outputs some information to the command line, which will be described in this section.
    
    1. Then you started the script successfully you should see in the command line the message `"Success connect to node!"`. If the connection could not be established, a message will be displayed on the screen `"Cannot connect to node! Check ws_node_url variable in .env file or asking to the developers team of the node..."`.
    2. Further, depending on which period you have chosen to claim rewards, the following commands will be displayed: for `SESSIONS` - `"Waiting for the end of the session..............."`, for `BLOCKS` - `"Waiting for the required number of blocks...............”`. If you made a mistake in writing the config and incorrectly specified the value of the variable `CLAIMING_TYPE`, you will see the message `"Unknown claiming_type! Check CLAIMING_TYPE var .env file"`.
    3. After the period specified in the configuration file has passed, you should see the following messages coming in a row `"Transaction submitted successfully!"` and `"claim_rewards success: Rewarded(...)"`. If this did not happen, then you will see one of the following error messages: `"You have no CGTs to pay the transaction fees!"` or `"You have no eligible to make the transaction! Firstly, check ACCOUNT_KEY var in .env file!"`. First means that you should add CGTs to your account and second one means, that you are not a collator or you have specified an incorrect private address in the configuration file.
    
    If you see errors of a different content in the command line, then contact the development team about them, who will be happy to help you solve the problem.
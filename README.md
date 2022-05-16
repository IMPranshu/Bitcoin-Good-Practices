# Bitcoin-Good-Practices

= Friday labs

Good sources for learning:
* Grokking Bitcoin
* https://bitcoin.org/en/full-node
* Googling
* Bitcoin wiki https://en.bitcoin.it/wiki

Load this document, https://rosenbaum.se/book/cc/instructions.txt, in a
web browser, so that you can refresh if I need to make changes to it.

=== If you are not running Linux

This document is a quick start guide for Linux, but much of it should
apply to Win and MacOS too. If you run Win or MacOS, I suggest that
you check with https://bitcoin.org/en/full-node for how to get started
on your OS. We are going to run Bitcoin Core in daemon mode, eg
without a GUI, so make sure you read the instructions for that. On
MacOS that involves downloading a different binary than the default
one, eg don't download the .dmg file, but the .tar.gz for MacOS.

== Part 1

I will go through and do Part 1 in the main room. When I'm done we'll
create breakout sessions where you do Part 1 yourselves. Discuss
issues with each other and ask and answer questions in your
group. I'll be available in the main room if there are any issues
along the way. Just come and get me (maybe I'm in another room at that
moment, but leave a message in the chat in that case).

=== Download, verify and run

# Get the latest version of bitcoin core for bitcoincore.org
$ wget https://bitcoincore.org/bin/bitcoin-core-0.21.1/bitcoin-0.21.1-x86_64-linux-gnu.tar.gz

# Get the corresponding signature file
$ wget https://bitcoincore.org/bin/bitcoin-core-0.21.1/SHA256SUMS.asc

# Verify that the hash of the tar.gz file matches the one
# in SHA256SUMS.asc
$ sha256sum --ignore-missing --check SHA256SUMS.asc

# Lookup the fingerprint of Bitcoin Core's release key.
# It should be 01EA5486DE18A882D4C2684590C8019E36C2E964
# But make sure that the fingerprint is authentic by
# checking with multiple sources. Example:
# Grokking Bitcoin
# Ask on forums
# Ask someone you trust
# Check bitcoincore.org
$ gpg --keyserver hkp://keyserver.ubuntu.com --recv-keys 01EA5486DE18A882D4C2684590C8019E36C2E964

# When you're sure you have the correct key, verify the signature in
# the signature file
$ gpg --verify SHA256SUMS.asc
# Look for "Good signature from..." and verify that the fingerprint matches what you expect

# Unpack Bitcoin Core
$ tar -zxvf bitcoin-0.21.1-x86_64-linux-gnu.tar.gz

$ cd bitcoin-0.21.1/bin

# Start Bitcoin Core in the background (-daemon)
# and run on a test network, called signet
$ ./bitcoind -signet -daemon

# Have a look at data from the blockchain
$ ./bitcoin-cli -signet getblockchaininfo
# Look for "initialblockdownload". True means it's sill
# syncing

# Stop Bitcoin Core
$ ./bitcoin-cli -signet stop

# Make it always run signet
$ echo "chain=signet" >> ~/.bitcoin/bitcoin.conf

# Enable transaction index, useful for the labs later
$ echo "txindex=1" >> ~/.bitcoin/bitcoin.conf

# Now you can start with only -daemon flag
$ ./bitcoind -daemon

# You can check again whether sync is finished
$ ./bitcoin-cli getblockchaininfo

=== Create, encrypt and backup wallet

# Use the help. It's very often needed
$ ./bitcoin-cli help

# Get help on a specific command
$ ./bitcoin-cli help createwallet

# Create a wallet
# Use -stdin to read arguments from standard input
$ ./bitcoin-cli -stdin createwallet grok false false
<password><ctrl-D><ctrl-D>

$ ./bitcoin-cli getwalletinfo

# Make a backup, note this doesn't use a mnemonic phrase
# which is ironic, since we've talked so much about that.
# Bitcoin core uses a simple copy of the wallet file for
# backups. For real Bitcoin you would store this and your
# wallet passphrase in a safe place.
$ ./bitcoin-cli backupwallet ~/walletbackup.dat

=== Receive and send some bitcoin

# Get a fresh address
$ ./bitcoin-cli getnewaddress
# When all participants of your group has an address
# compose a list with one address per line and
# send the list to Kalle on discord or zoom
# Kalle will send 5 TBTC (test bitcoin) to each address.
# Note: If you accidentally lose your coins due to
# mistakes, request new coins from your group or from
# Kalle.

# Get the confirmed balance
$ ./bitcoin-cli getbalance

# Get unconfirmed and confimed balances
$ ./bitcoin-cli getbalances
# See if you can figure out the three types
# of balances in the output. Use the help command.

# Get all transactions concerning your wallet
$ ./bitcoin-cli listtransactions

# To use your private keys, you must decrypt them. The following
# will decrypt and clean memory after 300 seconds.
$ ./bitcoin-cli -stdin walletpassphrase
<walletpassphrase>
300
<ctrl-d>

# Send some coins to a peer in your group
# Notice the -named flag which will require the arguments to
# be named this lets you specify only a selection of args
# that might be out of order and not consecutive.
$ ./bitcoin-cli -named sendtoaddress address=<address> amount=1

# Lock the wallet again. The 300 seconds is just a safeguard
# in case you forget to lock.
$ ./bitcoin-cli walletlock

== Part 2

Below are a few things you can do with your nodes. Choose any that
interests you and try to solve them in your group. Use the available
resources, "./bitcoin-cli help", and your group to solve the
assignments. I'll be available in the main room, just come and get me.

If you think the suggested assignments below are boring, you may come
up with your own ideas instead. Talk to me in that case. If the idea
is good, I'll add it to the list below.

While experimenting below, I suggest that you unlock your wallet for a
very long time so that you don't have to do it every time you need
your private keys. Also, some commands, eg the "send" command will
behave somewhat unintuitively if run while wallet is locked. So unlock
as follows:

$ ./bitcoin-cli -stdin walletpassphrase
<walletpassphrase>
50000
<ctrl-d>

1. Make multiple payments to different participants of your group in a
  single transaction. Each participant should do this, in which case
  each participant should create one address for each other
  participant to avoid address reuse.

2. Inspect a transaction using getrawtransaction (using verbose flag)
  or decoderawtransaction. Note that you must set txindex=1 in
  ~/.bitcoin/bitcoin.conf and restart your node to fetch arbitrary
  transactions. See if you can find the witness version and witness
  program of an output. Also, look at an input and explain to yourself
  and group members what your see.

3. Get a merkle proof ("txoutproof" in bitcoin-cli) for a transaction
  that proves that the transaction is included in a certain
  block. Send the proof to a group member so that he/she can verify
  it.

4. Fee bumping (see chapter 9). Send a transaction with a very low
  fee, 1 sat/vB (virtual byte, 1vB=4WU). Then immediately replace the
  transaction with a feature called "replace-by-fee" or "fee
  bumping". This will "double spend" and unconfirmed transaction with
  a conflicting transaction that has a higher fee. This can be used if
  your transaction doesn't confirm in reasonable time. see
  "bumpfee". Hint: a transaction needs to opt-in to replace-by-fee to
  be replacable.

5. Find out what the target is in one of your latest blocks. Note that
  since this is signet, the target will be much higher than on the
  main Bitcoin network. Note that difficulty and target are different
  measurements of the same thing, but we want the target.

  * Use section 11.3.2 to find the layout of a block header
  * Use the sidebar "Targets in Bitcoin" of section 7.3
  * The target is stored as little-endian so you have to reverse
    the bytes before using the "Targets in Bitcoin" sidebar.

  How many zero bits does it have in the beginning?

6. Create and send a transaction with an OP_RETURN (see chapter 9)
  output (called "data" output in bitcoin-cli). Lookup your
  transaction and inspect it using getrawtransaction with
  verbose=true. Hint: To encode a string to ascii code, use
  "printf '<a string>' | hexdump -C"

  Note: data outputs are limited to 80 bytes.

  When the transaction is confirmed, do

  $ strings ~/.bitcoin/signet/blocks/blk*.dat

  This searches all block data for strings and prints them. Can you
  find your string near the end?

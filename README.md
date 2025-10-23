TipJar Sui Move Module

This repository contains a simple Sui Move module for a tip_jar. This module allows any user to create a shared TipJar object that can receive SUI tips from anyone on the network. The tips are immediately transferred to the owner of the TipJar.

Features

Create a Tip Jar: Any user can publish this module and its init function will run, creating a new TipJar object owned by the publisher.

Shareable: The TipJar object is shared, making it accessible to everyone on the network.

Send Tips: Anyone can call the send_tip function to send SUI to a specific TipJar.

Direct Transfer: Tipped SUI (in the form of a Coin<SUI>) is directly transferred to the TipJar's owner.

Tracking: The TipJar object tracks the total amount of SUI received (total_tips_received) and the number of tips (tip_count).

Events: Emits events for TipJar creation and when a tip is sent.

Structs

TipJar

The main object that holds the state of a tip jar.

public struct TipJar has key {
    id: UID,
    owner: address,
    total_tips_received: u64,
    tip_count: u64,
}


id: The unique object ID of the TipJar.

owner: The address of the user who created the TipJar and receives all tips.

total_tips_received: A running total of all SUI (in MIST) tipped to this jar.

tip_count: A running count of the number of tips received.

Events

TipJarCreated

Emitted when a new TipJar is created via the init function.

public struct TipJarCreated has copy, drop {
    tip_jar_id: ID,
    owner: address,
}


TipSent

Emitted every time send_tip is successfully called.

public struct TipSent has copy, drop {
    tipper: address,
    amount: u64,
    total_tips: u64,
    tip_count: u64,
}


Public Functions

init(ctx: &mut TxContext)

The module initializer. It is called once when the module is published. It creates a new TipJar, emits a TipJarCreated event, and shares the TipJar object.

send_tip(tip_jar: &mut TipJar, payment: Coin<SUI>, ctx: &mut TxContext)

Allows anyone to send a tip to a TipJar.

tip_jar: A mutable reference to the TipJar object being tipped.

payment: The Coin<SUI> object being sent as a tip.

ctx: The transaction context, used to identify the tipper.

This function transfers the payment coin to the tip_jar.owner, increments the total_tips_received and tip_count, and emits a TipSent event.

Getter Functions

These are read-only functions to view the TipJar's state.

get_total_tips(tip_jar: &TipJar): u64
Returns the total_tips_received.

get_tip_count(tip_jar: &TipJar): u64
Returns the tip_count.

get_owner(tip_jar: &TipJar): address
Returns the owner's address.

is_owner(tip_jar: &TipJar, addr: address): bool
Returns true if the provided addr matches the tip_jar.owner.

Error Codes

EInvalidTipAmount (1):
Asserted in send_tip if the tip_amount is 0. The tip must be greater than 0.

Usage Example (Sui Client CLI)

Here is a step-by-step example of publishing and interacting with the TipJar module using the Sui client.

1. Check Initial Balance

First, check your address balance. You'll need SUI for gas fees to publish the module.

$ sui client balance
╭────────────────────────────────────────╮
│ Balance of coins owned by this address │
├────────────────────────────────────────┤
│ ╭─────────────────────────────────╮    │
│ │ coin   balance (raw)   balance    │    │
│ ├─────────────────────────────────┤    │
│ │ Sui    900000000       0.90 SUI   │    │
│ ╰─────────────────────────────────╯    │
╰────────────────────────────────────────╯


Here, the starting balance is 900,000,000 MIST, or 0.90 SUI.

2. Publish the Module

Next, run the publish command from your module's directory. This compiles, deploys, and runs the init function.

$ sui client publish


This will produce a large output, which is the transaction receipt.

3. Find Your New Objects

In the sui client publish output, look for the Object Changes or Created Objects section. This tells you what was created by your module's init function.

Example Output:

╭─────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                      │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                    │
│  ┌──                                                                                                │
│  │ ObjectID: 0x16a3e2aadb7b59ba56dfb5ff056fd0d26a136b8781917e536cabeb45019c40ab                       │
│  │ Sender: 0x6c5a9309b52408bb8fb815d21125ccdbe75b044c626877ef3da53c38671fdab3                       │
│  │ Owner: Account Address ( 0x6c5a9309b52408bb8fb815d21125ccdbe75b044c626877ef3da53c38671fdab3 )   │
│  │ ObjectType: 0x2::package::UpgradeCap                                                            │
│  └──                                                                                                │
│  ┌──                                                                                                │
│  │ ObjectID: 0x906c4af608b630e89317d6b0308c37d26f5ece5f53903ba8fad8c0023635f198                       │
│  │ Sender: 0x6c5a9309b52408bb8fb815d21125ccdbe75b044c626877ef3da53c38671fdab3                       │
│  │ Owner: Shared( 349180783 )                                                                        │
│  │ ObjectType: 0x9a651cec4d99234cd50984d818f4349bf20295e6f8cc3e9b6ab85c683aa92f79::tip_jar::TipJar    │
│  └──                                                                                                │
│  ┌──                                                                                                │
│  │ ObjectID: 0x9a651cec4d99234cd50984d818f4349bf20295e6f8cc3e9b6ab85c683aa92f79                       │
│  │ Sender: 0x6c5a9309b52408bb8fb815d21125ccdbe75b044c626877ef3da53c38671fdab3                       │
│  │ Owner: Immutable                                                                                │
│  │ ObjectType: 0x2::package::Package                                                               │
│  └──                                                                                                │


From this, you can identify three important objects:

Package ID: 0x9a651cec4d99234cd50984d818f4349bf20295e6f8cc3e9b6ab85c683aa92f79 (The one with ObjectType: 0x2::package::Package).

TipJar Object ID: 0x906c4af608b630e89317d6b0308c37d26f5ece5f53903ba8fad8c0023635f198 (The one with ObjectType: ...::tip_jar::TipJar and Owner: Shared).

Upgrade Cap: 0x16a3e2aadb7b59ba56dfb5ff056fd0d26a136b8781917e536cabeb45019c40ab (The one with ObjectType: 0x2::package::UpgradeCap).

You can also confirm the TipJar ID from the Transaction Block Events section:

│  │ EventType: 0x9a651cec4d99234cd50984d818f4349bf20295e6f8cc3e9b6ab85c683aa92f79::tip_jar::TipJarCreated │
│  │ ParsedJSON:                                                                                        │
│  │  ┌────────────┬────────────────────────────────────────────────────────────────────┐                │
│  │  │ owner      │ 0x6c5a9309b52408bb8fb815d21125ccdbe75b044c626877ef3da53c38671fdab3 │                │
│  │  ├────────────┼────────────────────────────────────────────────────────────────────┤                │
│  │  │ tip_jar_id │ 0x906c4af608b630e89317d6b0308c37d26f5ece5f53903ba8fad8c0023635f198 │                │
│  │  └────────────┴────────────────────────────────────────────────────────────────────┘                │


4. Send a Tip

Now that you have the Package ID and the TipJar Object ID, anyone can send a tip. You will also need the Object ID of a Coin<SUI> to use as payment.

# General command structure
$ sui client call --function send_tip \
    --module tip_jar \
    --package <PACKAGE_ID> \
    --args <TIP_JAR_ID> <COIN_TO_SEND_AS_TIP> \
    --gas-budget 30000000

# Using the IDs from our example:
$ sui client call --function send_tip \
    --module tip_jar \
    --package 0x9a651cec4d99234cd50984d818f4349bf20295e6f8cc3e9b6ab85c683aa92f79 \
    --args 0x906c4af608b630e89317d6b0308c37d26f5ece5f53903ba8fad8c0023635f198 0xCOIN_OBJECT_ID \
    --gas-budget 30000000


(Replace 0xCOIN_OBJECT_ID with an actual coin ID from your wallet)

5. Check Final Balance

After the publish transaction, your SUI balance will be lower because of gas fees.

Initial Balance: 900,000,000 MIST

Gas Cost Summary (from publish output):

Storage Cost: 13839600 MIST

Computation Cost: 1000000 MIST

Storage Rebate: -978120 MIST

Non-refundable Storage Fee: 9880 MIST

Total Gas Spent: 13,871,360 MIST (approx 0.0139 SUI)

New Balance (Approx): 900,000,000 - 13,871,360 = 886,128,640 MIST

If you run sui client balance again, you will see this new, lower balance.

$ sui client balance
╭────────────────────────────────────────╮
│ Balance of coins owned by this address │
├────────────────────────────────────────┤
│ ╭─────────────────────────────────╮    │
│ │ coin   balance (raw)   balance    │    │
│ ├─────────────────────────────────┤    │
│ │ Sui    886128640       0.88 SUI   │    │
│ ╰─────────────────────────────────╯    │
╰────────────────────────────────────────╯
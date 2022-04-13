# Build a Crowdfunding Module with Ignite

# Introduction

As blockchain technology gains more popularity and increasing adoption, it opens up a new era of crowdfunding. 

More people are openly funding open-source projects and it is easier than ever for builders to receive funding for their projects from across the globe irrespective of their location. This is something that is almost impossible with the traditional crowdfunding platforms.

In this tutorial, you will learn some basic crowdfunding model and use [Ignite](https://ignite.com/cli/) to build a crowdfunding module of your own. Your crowdfunding module will enable users create a crowdfunding campaign, pledge a donation, claim the donations if the goal is met, and also get refunds if the campaign's goal is not met.

Ignite makes it easier for you to create your own blockchain module without writing long lines of code. With just a single command, it will generate boilerplate code so you can focus on the application logic that makes your project unique. 

**By the end of this tutorial, you will be able to:**

* Scaffold a blockchain
* Scaffold a Cosmos SDK crowdfund module
* Scaffold a list for crowdfund objects
* Create messages in the crowdfund module to interact with the crowdfund object
* Interact with other Cosmos SDK modules
* Use an escrow module account
* Add application messages for a crowdfund system
* Create a crowdfunding campaign
* Donate to a crowdfunding campaign
* Withdraw funds from a crowdfunding campaign

# Prerequisites

This tutorial is written in Golang and assumes that you atleast have basic understanding of the language. However, if you do not have any experience writing Golang, you can take this beginner-friendly course from [Codecademy](https://www.codecademy.com/learn/learn-go) to get started.

Once you've done that, it'll be easier to understand this tutorial.

# Requirements

* **Golang**: Make sure you have Golang installed on your machine. If you don't already have it, you can install it by following the instructions [here](https://golang.org/doc/install). 

	You also need to be familiar with Golang syntax to follow along with this tutorial.
* **Ignite**: The Ignite CLI is responsible for scaffolding the blockchain and the crowdfund module. If you don't already have Ignite installed, you can run the folloowing command in your terminal.

	```text
	$ curl https://get.ignite.com/ignite! | bash
	```
	In case you encounter any errors during the installation, you can find a detailed installation guide [here](https://docs.ignite.com/guide/install.html).

Now that you are fully setup with the prerequisites, you can proceed to the next section.

# Getting Started

## Terminologies

During the course of this tutorial, we will be mentioning a few terminologies that may be alien to you. Therefore, we will be explaining them here to make the tutorial easier to follow along with.

### Modules

A module is a collection of related functions that can be used in a blockchain application. Blockchain modules follow the same conventions as the traditional modules in other languages.

The crowdfunding module, for instance, is just a set of functions that can be used to execute a crowdfunding campaign.

Modules contain one or more [`Messages`](#messages).

### Messages

Messages are objects that trigger state-transitions. They are functions inside a module.

However, unlike in conventional modular programming architecture, messages are functions of functions. They can be executed by sending a transaction to the module.

### Scaffold

When you scaffold any part of an application, you are automatically generating the code that is required for it to run. Instead of spending hours writing so much code from scratch, Ignite will automatically generate the code for you.

### Transactions

Transactions are objects created by end-users to trigger state changes in the application (e.g. creating a gofundme campaign, making a donation to the campaign, etc.). When a user sends a transaction, they are sending a [`message`](#messages) to the application.

This is similar to calling an application's function. 

## Module Design

Before going into the actual tutorial, let's take a look at the design of the crowdfund module.

The crowdfunding campaign consists of: 

* An `id`
* The `goal` of the campaign
* The `start` and `end` of the campaign
* The `totaldonations` made towards the campaign
* The `claim` of the campaign that describes the status as:
    * `true` if the donations have been withdrawn
    * `false` if the donations have not been withdrawn
* The amount of `donation` made by an individual to the campaign
* The `donor` of the donation
* The `creator` of the campaign

### The Creator

A Creator creates a crowdfunding campaign with the following information:

* `goal`
* `start`
* `end`

The Creator can create a campaign and also donate to the campaign. However, the Creator can only withdraw funds from the campaign if the goal was met before the `end` date. 

### The Donor

A Donor donates to a crowdfunding campaign with the following information:

* `id` of the campaign
* `donation` amount

If the goal of the campaign is not met before the `end` date, the `Donor`(s) can withdraw the(ir) `donation`(s) from the campaign.

Great! Now let's get started with the actual implementation.

## Scaffold the Blockchain 

Use Ignite to scaffold a fully functional Cosmos SDK blockchain app named `gofundme` with the following command.

```text
ignite scaffold chain github.com/cosmonaut/gofundme --no-module
```

The `--no-module` flag prevents scaffolding a default module. Don't worry, you will add the `gofundme` module later.

Go into the newly created `gofundme` directory:

```text
cd gofundme
``` 
## Building a Blockchain Module

Scaffold the module to create a new `gofundme` module. Following the Cosmos SDK convention, all modules are scaffolded inside the `x` directory:

```text
ignite scaffold module gofundme --dep bank
```

Use the `--dep` flag to specify that this module depends on and is going to interact with the Cosmos SDK `bank` module.

You can think of this like importing a package or library into your project. Once you add the `bank` module as a dependency, you can use the `bank` module's functions in your `gofundme` module.

## Scaffolding a List

Use the [scaffold list](https://docs.ignite.com/cli/#ignite-scaffold-list) command to scaffold code necessary to store `gofundme` campaigns in an array-like data structure:

```text
ignite scaffold list gofundme creator goal:uint start end totaldonations:uint claim:bool donation:array.string donor:array.string --no-message
```

By default, each variable defined in the list is a `string` type. You can change the type of the variable by using the `:` flag. For example, to change the type of the `goal` variable to `uint`, use the `goal:uint` flag.

Make `donor` and `donation` variables an array of string using the `:array.string` flag. This is necessary to store all the `donor`s and their `donation` information in an array.

Use the `--no-message` flag to disable CRUD messages in the scaffold.

The data you store in an array-like data structure are the `gofundme` campaigns. You can see the parameters defined above in the `Gofundme` message in `proto/gofundme/gofundme.proto`:

```text
message Gofundme {
  uint64 id = 1;
  string creator = 2; 
  uint64 goal = 3; 
  string start = 4; 
  string end = 5; 
  uint64 totaldonations = 6; 
  bool claim = 7; 
  repeated string donation = 8; 
  repeated string donor = 9; 
  
}
```

Now, it is time to use messages to interact with the gofundme module. But first, make sure to store your current state in a git commit:

```text
git add .
git commit -m "Scaffold gofundme and list modules" 
```

## Scaffolding the Messages

In order to create a gofundme app, you need the following messages:

* Create Gofundme
* Donate Fund
* Withdraw Donation

You can use the `ignite scaffold message` command to create each of the messages. Meke sure you define the details of each message when you scaffold them.

Create the messages one at a time with the following application logic.

### Create a Gofundme Campaign

The initial message handles the transaction that occurs when an account (or user) creates a gofundme campaign.

In this case, the user needs a certain amount of funds and is requesting donations from other users.

Therefore, the first message is the `create-gofundme` message, which requires the following input parameters:

* `goal`: The amount of funds the user is requesting
* `start`: The start date of the gofundme campaign
* `end`: The end date of the gofundme campaign

Create a message with the following command:

```text
ignite scaffold message create-gofundme goal:uint start end 
```
Define the `goal` as a `uint` and the `start` and `end` as `string` types.

The `create-gofundme` message creates a new campaign and stores it in the `gofundme` list. Describe the logic of the message in `x/gofundme/keeper/msg_server_create_gofundme.go`.

```golang
package keeper

import (
	"context"

	"github.com/cosmonaut/gofundme/x/gofundme/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
)

func (k msgServer) CreateGofundme(goCtx context.Context, msg *types.MsgCreateGofundme) (*types.MsgCreateGofundmeResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)

	// Start a gofundme with the following user inputs
	var gofundme = types.Gofundme{
		Creator: msg.Creator,
		Goal:    msg.Goal,
		Start:   msg.Start,
		End:     msg.End,
		Claim:   false,
	}

    //Allow users to set their gofundme campaign start date to be current time
	if gofundme.Start == "now" {
		gofundme.Start = ctx.BlockTime().Format("2006-01-02T15:04:05.000000000")
	}

	// Add the gofundme to the keeper
	k.AppendGofundme(ctx, gofundme)

	return &types.MsgCreateGofundmeResponse{}, nil
}
```
When a `gofundme` campaign is created, a certain input validation is required. You want to throw error messages in case the end user tries impossible inputs.

You can describe message validation errors in the modules types directory.

Add the following code to the ValidateBasic() function in the `/x/gofundme/types/message_create_gofundme.go` file:

```golang
func (msg *MsgCreateGofundme) ValidateBasic() error {
	// Validate the creator
	_, err := sdk.AccAddressFromBech32(msg.Creator)
	if err != nil {
		return sdkerrors.Wrapf(sdkerrors.ErrInvalidAddress, "invalid creator address (%s)", err)
	}

	// Get the goal amount
	goal := sdk.NewIntFromUint64(msg.Goal)

	// Ensure that the goal amount is greater than 0
	if goal.IsZero() {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "goal amount must be greater than 0")
	}

	var start time.Time
	//Get the start date
	if msg.Start != "now" {
		startDate, err := sdk.ParseTimeBytes([]byte(msg.Start))
		if err != nil {
			return err
		}
		start = startDate
	} else {
		start = time.Now()
	}

	//Get the end date
	end, err := sdk.ParseTimeBytes([]byte(msg.End))
	if err != nil {
		return err
	}

	// Get the current block time
	blockTime := time.Now()

	// Ensure that end date is in the future
	if end.Before(blockTime) {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "end date must be in the future")
	}

    //Ensure that start date is not in the past
	if start.Before(blockTime) {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "start date cannot be in the past")
	}

	// Ensure that start date is before end date
	if start.After(end) {
		return sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "start date must be before end date")
	}
	return nil
}

```
Congratulations, you have created the `create-gofundme` message.

You can run the chain and test your first message.

Open a new CLI to start the blockchain:
```text
ignite chain serve
```
Create your first gofundme campaign:
```text
gofundmed tx gofundme create-gofundme 100 2022-04-02T15:04:05.000000000 2022-05-02T15:04:05.000000000 --from alice
```
You can also experiment with invalid inputs to see how the validator will throw errors.

Query your gofundme campaign:
```text
gofundmed query gofundme list-gofundme
```
The output should be something like this:
```text
Gofundme:
- claim: false
  creator: cosmos180m28ygyt3gapllfq0mgy28na6uzfs9hjggk9f
  donation: []
  donor: []
  end: 2022-05-02T15:04:05.000000000
  goal: "100"
  id: "0"
  start: 2022-04-02T15:04:05.000000000
  totaldonations: "0"
pagination:
  next_key: null
  total: "0"
```
If you encounter any error, you can restart your chain and try again.

You can stop the blockchain again with `CTRL+C` or `CMD+C`.

This is a good time to add your advancements to git:

```text
git add .
git commit -m "Scaffold create-gofundme message"
```

### Donate to Gofundme Campaign

After a gofundme campaign is created, other cosmonauts can donate to it. 

The next message is the `donate-fund` message, which requires the following input parameters:

* `id`: The id of the gofundme campaign
* `donation`: The amount of funds an account is donating

Create the `donate-fund` message with the following command:

```text
ignite scaffold message donate-fund id:uint donation
```
`id`s are stored as a `uint` by default. Therefore, you need to define the `id` as a `uint` type.

This message adds a new donation to the `gofundme` campaign. Describe the logic of the message in `x/gofundme/keeper/msg_server_donate_fund.go`.

```golang
package keeper

import (
	"context"

	"github.com/cosmonaut/gofundme/x/gofundme/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

func (k msgServer) DonateFund(goCtx context.Context, msg *types.MsgDonateFund) (*types.MsgDonateFundResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)

    //Find a gofundme campaign using its id
	gofundme, found := k.GetGofundme(ctx, msg.Id)
	if !found {
		return nil, sdkerrors.Wrapf(sdkerrors.ErrKeyNotFound, "gofundme campaign with id %s not found", msg.Id)
	}

	//Get the start date
	start, err := sdk.ParseTimeBytes([]byte(gofundme.Start))
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, err.Error())
	}

	//Get the end date
	end, err := sdk.ParseTimeBytes([]byte(gofundme.End))
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, err.Error())
	}

	//Check whether the current time is between the start and end date
	if ctx.BlockTime().Before(start) || ctx.BlockTime().After(end) {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "gofundme campaign is not active")
	}

	//Get the donor's address
	donor, err := sdk.AccAddressFromBech32(msg.Creator)
	if err != nil {
		return nil, sdkerrors.Wrapf(sdkerrors.ErrInvalidAddress, "invalid donor address (%s)", donor)
	}
	//Get the donation amount
	donation, _ := sdk.ParseCoinsNormalized(msg.Donation)

	//Ensure that the donation amount is greater than 0
	if donation.IsZero() {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "donation amount must be greater than 0")
	}

	//Update the gofundme status
	gofundme.Donor = append(gofundme.Donor, msg.Creator)
	gofundme.Donation = append(gofundme.Donation, msg.Donation)
	gofundme.Totaldonations += donation[0].Amount.Uint64()

	//Use the module account as an escrow account
	sdkError := k.bankKeeper.SendCoinsFromAccountToModule(ctx, donor, types.ModuleName, donation)
	if sdkError != nil {
		return nil, sdkError
	}

	k.SetGofundme(ctx, gofundme)

	return &types.MsgDonateFundResponse{}, nil
}
```
The module uses the `SendCoinsFromAccountToModule` function of `bankKeeper`. Add this `SendCoinsFromAccountToModule` function to the `x/gofundme/types/expected_keepers.go` file to get rid of the error.

```golang
type BankKeeper interface {
	SpendableCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
	SendCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
}
```
Restart the blockchain and use the two commands you already have available:

```text
ignite chain serve -r
```
Use the `-r` flag to reset the blockchain state and start with a new database.

Now, let `alice` create a gofundme campaign:
```text
gofundmed tx gofundme create-gofundme 100 now 2022-05-02T15:04:05.000000000 --from alice
```
Start the campaign immediately by setting the `start` date to `now`.

Query your gofundme campaign:
```text
gofundmed query gofundme list-gofundme
```

Donate to `alice`'s gofundme campaign:
```text
gofundmed tx gofundme donate-fund 0 100token --from bob
```
This donates 100 tokens to `alice`'s gofundme campaign.

Query the gofundme campaign again:
```text
Gofundme:
- claim: false
  creator: cosmos16fjh2va2drgnulxcnpeqcu6wzmhsajgwx3syqy
  donation:
  - 100token
  donor:
  - cosmos16usfuwy7ggaqua0k9ey2rmws09xgyr090yyrdx
  end: 2022-05-02T15:04:05.000000000
  goal: "100"
  id: "0"
  start: 2022-03-15T14:56:51.559035000
  totaldonations: "100"
pagination:
  next_key: null
  total: "0"
```
You see that `totaldonations` is now 100. The `donor` and `donation` lists have also been updated.

Play around with it and try donating different amounts to see how the total donations change.

This is a good time to save the state with a git commit:

```text
git add .
git commit -m "Add donate-fund message"
```

### Withdraw Donation Message

After the campaign is over, you can withdraw the donations. If the campaign goal is met, the creator of the campaign can withdraw the donations. Otherwise, donors can initiate a refund.

Scaffold the message `withdraw-donation` with the following command:
```text
ignite scaffold message withdraw-donation id:uint
```

Withdrawing donations requires that the campaign is over and `claim` is set to `no`. If these conditions are met, then `donation` is released from the escrow module account.

The withdrawal logic is defined in the `x/gofundme/keeper/message_server_withdraw_donation.go` file.

```golang
package keeper

import (
	"context"
	"fmt"

	"strconv"

	"github.com/cosmonaut/gofundme/x/gofundme/types"
	sdk "github.com/cosmos/cosmos-sdk/types"
	sdkerrors "github.com/cosmos/cosmos-sdk/types/errors"
)

func (k msgServer) WithdrawDonation(goCtx context.Context, msg *types.MsgWithdrawDonation) (*types.MsgWithdrawDonationResponse, error) {
	ctx := sdk.UnwrapSDKContext(goCtx)

    //Get the gofundme campaign with the given id
	gofundme, found := k.GetGofundme(ctx, msg.Id)
	if !found {
		return nil, sdkerrors.Wrapf(sdkerrors.ErrKeyNotFound, "gofundme with id %s not found", msg.Id)
	}

	//Confirm that the donations have not been claimed
	if gofundme.Claim == true {
		return nil, sdkerrors.Wrapf(sdkerrors.ErrUnauthorized, "gofundme with id %s has already been claimed", msg.Id)
	}

	//Get the end date
	end, err := sdk.ParseTimeBytes([]byte(gofundme.End))
	if err != nil {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Can't withdraw donations until the end of the gofundme campaign.")
	}

	//Check whether the current time is after the end date
	if ctx.BlockTime().Before(end) {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "gofundme campaign is not over. Please wait until the end date")
	}

	//Get gofundme creator and message sender
	creator := sdk.AccAddress(gofundme.Creator)
	messageSender := sdk.AccAddress(msg.Creator)

	//Get total donations and gofundme goal
	totalDonations := gofundme.Totaldonations
	goal := gofundme.Goal
	stringTotalDonation := strconv.FormatUint(totalDonations, 10)
	normalizedTotalDonation, _ := sdk.ParseCoinsNormalized(stringTotalDonation)
	fmt.Print(creator, messageSender)

	//Ensure that creator can withdraw only if the goal was met
	if creator.Equals(messageSender) && totalDonations < goal {
		return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Can't withdraw donations because goal wasn't met.")
	}

	if creator.Equals(messageSender) && totalDonations >= goal {
        //update claim to true
		gofundme.Claim = true
		sdkError := k.bankKeeper.SendCoinsFromModuleToAccount(ctx, types.ModuleName, creator, normalizedTotalDonation)
		if sdkError != nil {
			return nil, sdkError
		}
	}

	//Ensure that donor can withdraw only if goal wasn't met
	if !creator.Equals(messageSender) && totalDonations >= goal {
		return nil, sdkerrors.Wrap(sdkerrors.ErrUnauthorized, "Only the owner of the crowdfund can withdraw donations")
	}

	if !creator.Equals(messageSender) && totalDonations < goal {
		var index int
		for idx, v := range gofundme.Donation {
			if v == msg.Creator {
				index = idx
				break
			}
		}
		donation := gofundme.Donation[index]
		normalizedDonation, _ := sdk.ParseCoinsNormalized(donation)

		//Ensure that donor can only withdraw once
		if normalizedDonation.IsZero() {
			return nil, sdkerrors.Wrap(sdkerrors.ErrInvalidRequest, "Donation already withdrawn")
		}
		//Update the donation amount to zero to reflect that the donation was withdrawn
		gofundme.Donation[index] = "0"
		sdkError := k.bankKeeper.SendCoinsFromModuleToAccount(ctx, types.ModuleName, messageSender, normalizedDonation)
		if sdkError != nil {
			return nil, sdkError
		}
	}

	k.SetGofundme(ctx, gofundme)

	return &types.MsgWithdrawDonationResponse{}, nil
}

```

The module uses the `SendCoinsFromModuleToAccount` function of `bankKeeper`. Add this `SendCoinsFromModuleToAccount` function to the `x/gofundme/types/expected_keepers.go` file to get rid of the error.

```golang
type BankKeeper interface {
	SpendableCoins(ctx sdk.Context, addr sdk.AccAddress) sdk.Coins
	SendCoinsFromAccountToModule(ctx sdk.Context, senderAddr sdk.AccAddress, recipientModule string, amt sdk.Coins) error
	SendCoinsFromModuleToAccount(ctx sdk.Context, senderModule string, recipientAddr sdk.AccAddress, amt sdk.Coins) error
}
```
Now, restart the blockchain again to test all your code.
```text
ignite chain serve -r
```
Create a new gofundme campaign with the following command:
```text
gofundmed tx gofundme create-gofundme 100 now 2022-05-02T15:04:05.000000000 --from alice
```
Start the campaign immediately by setting the `start` date to `now`. to test this out immediately, ensure that the `end` date is only a few minutes from now.

Query your gofundme campaign:
```text
gofundmed query gofundme list-gofundme
```

Donate to `alice`'s gofundme campaign:
```text
gofundmed tx gofundme donate-fund 0 100token --from bob
```
This donates 100 tokens to `alice`'s gofundme campaign.

Query the gofundme campaign again:
```text
Gofundme:
- claim: false
  creator: cosmos16fjh2va2drgnulxcnpeqcu6wzmhsajgwx3syqy
  donation:
  - 100token
  donor:
  - cosmos16usfuwy7ggaqua0k9ey2rmws09xgyr090yyrdx
  end: 2022-05-02T15:04:05.000000000
  goal: "100"
  id: "0"
  start: 2022-03-15T14:56:51.559035000
  totaldonations: "100"
pagination:
  next_key: null
  total: "0"
```

Withdraw your donations:
```text
gofundmed tx gofundme withdraw-donation 0 --from alice
```
Query the gofundme campaign again:
```text
Gofundme:
- claim: true
  creator: cosmos176zax4hdqrpda3qcv8h2vwhsugc7u64z4nvs77
  donation:
  - 100token
  donor:
  - cosmos1zajjgdkx8jgckrpja0a2qhl853sjf0x25py4cl
  end: 2022-03-15T16:53:05.000000000
  goal: "100"
  id: "0"
  start: 2022-03-15T15:52:43.369278000
  totaldonations: "100"
pagination:
  next_key: null
  total: "0"
```
You see that `claim` is now `true`. If you try withdrawing your donations again, you will get an error.

Play around with it a little more to see how the validations work.

Update your local repository again with a git commit. After you test and use your loan module, consider publishing your code to a public repository for others to see your accomplishments.

```text
git add .
git commit -m "Add withdraw-donation module"
```

# Conclusion

Congratulations! You have completed the gofundme module tutorial.

You executed commands and updated files to:

* Scaffold a blockchain
* Scaffold a module
* Scaffold a list for gofundme objects
* Create messages in your module to interact with the gofundme object
* Interact with other modules in your module
* Use an escrow module account
* Add application messages for a gofundme system
    * Create Gofundme
    * Donate Fund
    * Withdraw Donation

The complete code for this tutorial can be found on [Github](https://github.com/jelilat/gofundme)

**Note:** *The code in this tutorial is written specifically for this learning experience and is intended only for educational purposes. This tutorial code is not intended to be used in production.*

# About the Author

This tutorial was created by [Jelilat Anofiu](https://github.com/jelilat/). If you'd like to stay up-to-date with my tutorials, follow me on [Medium](https://medium.com/@jelilah/) and [@tjelailah](https://twitter.com/tjelailah) on Twitter.

# Create a custom subnet : Avalanche Hypersdk

This project demonstrate the install of avalanche go and using avalanche Hypersdk to create virtual machine and subnet on avalanche platform. This allow developer to build ans customize tailored to specific use cases, such as mint,transfer,assert trading, check balance.

## Discription

This project allow user to install GO and write different command on terminal to set up  the virtual machine and perform different task by writing down the command of to create the asset , mint the asset, view the balance present on asset which i have set while minting the asset, create order , fill part of order and etc.

## Getting Start

### Installing
-> Install GO by using this link https://go.dev/doc/install

## Execute Program

->  Normalize all dependencies by runing this (go mod tidy) command on terminal
-> export PATH=$PATH:$(go env GOPATH)/bin ru this command to make sure Go is in path
-> run this MODE="run-single" ./scripts/run.sh to pass all aftersuite and beforesuite
-> run this ./scripts/build.sh
-> load private key using this command ./build/token-cli key import demo.pk
-> cuild token chain using this command ./build/token-cli chain import-anr

Since HyperSDK is alpha software, we are using the example token VM, which is one of the few that has been tested and guaranteed to be maintained in the upcoming future. Consult the repo constantly to keep your project up to date.

### Installation

1. **Clone the Repository:**

   ```bash
   git clone <repository_url>
   cd <repository_directory>

2. **Normalize Dependencies:**

   ```bash
   go mod tidy

### Configure Project Constants:


Edit `consts/consts.go` (same as below) to define the constants and initialize necessary values for your project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package consts

import (
	"github.com/ava-labs/avalanchego/ids"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"
	"github.com/ava-labs/hypersdk/consts"
)

const (
	// TODO: choose a human-readable part for your hyperchain
	HRP = "thypersdk"
	// TODO: choose a name for your hyperchain
	Name = "Amrita"
	// TODO: choose a token symbol
	Symbol = "Tuhi"
)

var ID ids.ID

func init() {
	b := make([]byte, consts.IDLen)
	copy(b, []byte(Name))
	vmID, err := ids.ToID(b)
	if err != nil {
		panic(err)
	}
	ID = vmID
}

// Instantiate registry here so it can be imported by any package. We set these
// values in [controller/registry].
var (
	ActionRegistry *codec.TypeParser[chain.Action, *warp.Message, bool]
	AuthRegistry   *codec.TypeParser[chain.Auth, *warp.Message, bool]
)
```
### Register Actions:


Edit `registry/registry.go` (same as below) to register actions and initialize registries for your custom blockchain project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package registry

import (
	"github.com/ava-labs/avalanchego/utils/wrappers"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"

	"tokenvm/actions"
	"tokenvm/auth"
	"tokenvm/consts"
)

// Setup types
func init() {
	consts.ActionRegistry = codec.NewTypeParser[chain.Action, *warp.Message]()
	consts.AuthRegistry = codec.NewTypeParser[chain.Auth, *warp.Message]()

	errs := &wrappers.Errs{}
	errs.Add(
		// When registering new actions, ALWAYS make sure to append at the end.
		consts.ActionRegistry.Register(&actions.Transfer{}, actions.UnmarshalTransfer, false),
		consts.ActionRegistry.Register(&actions.CreateAsset{}, actions.UnmarshalCreateAsset, false),
		consts.ActionRegistry.Register(&actions.MintAsset{}, actions.UnmarshalMintAsset, false),
		consts.ActionRegistry.Register(&actions.BurnAsset{}, actions.UnmarshalBurnAsset, false),
		consts.ActionRegistry.Register(&actions.ModifyAsset{}, actions.UnmarshalModifyAsset, false),

		consts.ActionRegistry.Register(&actions.CreateOrder{}, actions.UnmarshalCreateOrder, false),
		consts.ActionRegistry.Register(&actions.FillOrder{}, actions.UnmarshalFillOrder, false),
		consts.ActionRegistry.Register(&actions.CloseOrder{}, actions.UnmarshalCloseOrder, false),

		consts.ActionRegistry.Register(&actions.ImportAsset{}, actions.UnmarshalImportAsset, true),
		consts.ActionRegistry.Register(&actions.ExportAsset{}, actions.UnmarshalExportAsset, false),

		// When registering new auth, ALWAYS make sure to append at the end.
		consts.AuthRegistry.Register(&auth.ED25519{}, auth.UnmarshalED25519, false),
	)
	if errs.Errored() {
		panic(errs.Err)
	}
}
```
### Run the Virtual Machine Locally:

Ensure Go is on your path. If not, run:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```
### Execute the following commands to run and build the VM:

```bash
MODE="run-single" ./scripts/run.sh
./scripts/build.sh
```
### Load Demo Private Key:

Import the demo private key included in the project:

```bash
./build/token-cli key import demo.pk
./build/token-cli chain import-anr
```
### Interact with Your HyperChain:

### Mint and Trade
#### Step 1: Create Your Asset
First up, let's create our own asset. You can do so by running the following
command from this location:
```bash
./build/token-cli action create-asset
```

When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
metadata (can be changed later): tcoin
continue (y/n): y
✅ txID: UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
```
_`txID` is the `assetID` of your new asset._

The "loaded address" here is the address of the default private key (`demo.pk`). We
use this key to authenticate all interactions with the `tokenvm`.

#### Step 2: Mint Your Asset

After we've created our own asset, we can now mint some of it. You can do so by
running the following command from this location:
```bash
./build/token-cli action mint-asset
```

When you are done, the output should look something like this (usually easiest
just to mint to yourself).
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
assetID: UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
metadata: tcoin supply: 0
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 1000
continue (y/n): y
✅ txID: 2SdapZvHZX3QrbNSWBXp9yVQV1soQzTjX4vFJ2XW7pYD6jDaTJ
```

### Transfer Assets to Another Subnet

Unlike the mint and trade demo, the AWM demo only requires running a single
command. You can kick off a transfer between the 2 Subnets you created by
running the following command from this location:

```bash
./build/token-cli action export
```

When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
✔ assetID (use TKN for native token): SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
balance: 1000 SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 10
reward: 0
available chains: 1 excluded: [Em2pZtHr7rDCzii43an2bBi1M2mTFyLN33QP1Xfjy7BcWtaH9]
0) chainID: cKVefMmNPSKmLoshR15Fzxmx52Y5yUSPqWiJsNFUg1WgNQVMX
destination: 0
swap on import (y/n): n
continue (y/n): y
✅ txID: 2SdapZvHZX3QrbNSWBXp9yVQV1soQzTjX4vFJ2XW7pYD6jDaTJ
perform import on destination (y/n): y
2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf to: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp source assetID: TKN output assetID: UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT value: 10000000000 reward: 10000000000 return: false
✔ switch default chain to destination (y/n): y
```

_The `export` command will automatically run the `import` command on the
destination. If you wish to import the AWM message using a separate account,
you can run the `import` command after changing your key._

### Create order

for creating order use this command

```
bash
./build/token-cli action create-order
```

when done the output should look like
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
in assetID (use TKN for native token): UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
metadata: tcoin supply: 1000 warp: false
in tick: 1
out assetID (use TKN for native token): UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
metadata: tcoin supply: 1000 warp: false
balance: 1000 UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
out tick: 10
supply (must be multiple of out tick): 100
continue (y/n): y
⚠️ txID: 2LW1cT8GFoKFxd3ZxcvgqoEX2iUBuWc4xkZZAzERsFknz7SaBQ
```


### for filling order

for filling order use this command

```
bash
./build/token-cli action fill-order
```

when done the output should look like
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
in assetID (use TKN for native token): UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
metadata: tcoin supply: 1000 warp: false
balance: 1000 UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
out assetID (use TKN for native token): UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
metadata: tcoin supply: 1000 warp: false
available orders: 1
continue (y/n): y
⚠️ txID: 2LW1cT8GFoKFxd3ZxcvgqoEX2iUBuWc4xkZZAzERsFknz7SaBQ
```

### closing order

for creating order use this command

```
bash
./build/token-cli action close-order```

when done the output should look like
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
orderID: 2WmuhnA8JB2qydZC5SSvddc1YjN2nhfWJzSi3aGH15iiMM73rf
out assetID (use TKN for native token): UQRwwa6Y4iPTaKVoZyR9UEoGUkQCSojM6aQnDGu6vnSWsG2TT
continue (y/n): y
⚠️ txID: 233NQtRDsUu9URdc5s3E4vtoRTC9kvFF2Wjq8AvarTZFtcpVaH
```

### Closing the Local Avalanche Network:
To shut down the local Avalanche network, run:

```bash
killall avalanche-network-runner

```
### CONCLUSION
- So we have created a custom virtual machine to enable users to mint and transfer tokens.
- With the HyperSDK, we have defined the rules and functionality of our chain, including the ability to create, mint and transfer tokens .
  
## Authors

Amrita kumari

https://www.linkedin.com/in/amrita-kumari-753319232/

## License

This project is licensed under the MIT License - see the LICENSE.md file for details.

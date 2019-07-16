---
layout: post
title:  "Demo of CYW"
date:   2019-07-14 08:17:43 +0530
categories: jekyll update
---

Before going into the heart of the matter, we create an instance of all the smart contracts

```javascript
 truffle(ganache)> DumpToken.deployed().then(instance=>{dumpToken=instance})
undefined
truffle(ganache)> SmartGarbageBin.deployed().then(instance=>{smartGarbageBin=instance})
undefined
truffle(ganache)> VehicleManagement.deployed().then(instance=>{vehicleManagement=instance})
undefined

```

Next set up ten accounts with public addresses. We also set up te address of the owner

```javascript
truffle(ganache)> var accounts = await web3.eth.getAccounts()
undefined
truffle(ganache)> accounts
[ '0xd4a78EC056C9D715b0d06640C9372e070FB29552',
  '0x11490C3908517b6FDAfc81BDC178c3DC8971DAa8',
  '0x2923eB794182117beBE4919dd6047A0280d3eBE8',
  '0xe07505BF86a2A0eb8EB93aedF88627773C142774',
  '0xA06cbF9263e787B0908F752D40636023384fB81b',
  '0xa488CcBB600be7f53B8b7fC1bd94F859ddf112B4',
  '0x164F32B747262e5dd3226a3FF8642e0fA6b9551E',
  '0x35968fA53A7CD532502cC905e047e49573b11C8a',
  '0x805154965035df30EC63DC0879C3fD8ab55a1e8a',
  '0x7e2709F9Db03a6e5f74ed16b7712f7A69669eeFF' ]
truffle(ganache)> var owner = accounts[0]
undefined
truffle(ganache)> owner
'0xd4a78EC056C9D715b0d06640C9372e070FB29552'
```
Now we send 1000000 XDT to DumpToken contract account has thus giving it the following order
(A) Initial holder of XDT
and (B) Decentralized distributor of XDT. 


Event Triggered : Transfer.

```javascript
truffle(ganache)> dumpToken.transfer(dumpToken.address, 100000)
{ tx:
   '0x80ae2dff922002dbdcc33d6b2c08fad38a6b4662aead2034cfa725500094b157',
   				.
   				.
   				.
   	event: 'Transfer',
       args: [Result] } ] }
```			
Now lets make some assumptions

Assumption :  User has responsibly dumped the waste [ wasteDump(Dump request from user) ].

Next, [F1] Smart Garbage Bin detects the Waste is responsibly dumped [ SGBSensing() ] and invokes the [F2] smart contract to transfer incentive to the user [ transferIncentive(User ID, amount) ] as shown in the code snippet below 

Event Triggered: TransferIncentive.

```javascript
var user = accounts[1]
undefined
truffle(ganache)> dumpToken.transferIncentive(user, 10)
{ tx:
   '0xa086565a5b58e54f1f872326d4128e4f596237b6462db44d32a10d58ec2c2044',
   				.
   				.
   				.
   	event: 'TransferIncentive',
       args: [Result] } ] }
```

Assumption: The Smart Garbage Bin is full.

[F3] This new state of the Smart Garbage Bin is checked [ SGBMessaging() ] and [F4] recorded on the blockchain by calling the smart contract [ setSmartGarbageBinStatus() ]
```javascript
truffle(ganache)> var SGB_ID = "S3"
undefined
truffle(ganache)> var SGB_Loc = "Rabindra Sarobor"
undefined
truffle(ganache)> var SGB_Status = "FULL"
undefined
truffle(ganache)> smartGarbageBin.setSmartGarbageBinStatus(SGB_ID, SGB_Loc, SGB_Status)
{ tx:
   '0xb2323a8732caa5f2754c4d60c849c1dde95e23fde3e6c6f45487fa5ae3735874',
   						.
   						.
   	event: 'SGB_Update',
       args: [Result] } ] }
```

[F5] Next, the nearby vehicle are calcuted for pick up notification [ SGBNotifying() ] and [F6] the pick up notification is sent to the nearby vehicles by calling the smart contract [ sendSGBstatus() ] as shown in the code snippet below

Events Triggered: NearbyVehiclesNotification, PendingRequest
```javascript
var nearbyVehicles = [ accounts[5], accounts[6] ]
undefined
truffle(ganache)> vehicleManagement.sendSmartGarbageBinStatus(SGB_ID, SGB_Loc, SGB_Status, nearbyVehicles[0])
{ tx:
   '0xbc8e178d200f784fec58c29b0808785d37507b34243e4ae739384a0869dc8ed3',
   						.
   						.
	event: 'NearbyVehiclesNotification',
        args: [Result] },
    					.
    					.
    event: 'PendingRequest',
       args: [Result] } ] }
```
					
[F7] The nearby vehicles send their response [ sendResponse()] by  [F8] calling the smart contract [ receiveResponse()] with their response.

Events Triggered: RespondToPickup, PendingRequest

```javascript
var response = "AVAILABLE"
undefined
truffle(ganache)> vehicleManagement.receiveResponse(response, nearbyVehicles[0])
{ tx:
   '0xca4e99b9085bbd68d3f6c67e5e5fe2476af1dbb015fa2dd73a49763a8d86011f',
   						.
   						.
   	event: 'RespondToPickup',
    args: [Result] },
    					.
    					.
    event: 'PendingRequest',
       args: [Result] } ] }

truffle(ganache)> vehicleManagement.receiveResponse(response, nearbyVehicles[1])
{ tx:
   '0xa740b2e50acd3ccd09bd4e40dd8447bdd29d0d330ac18db491366b10e8b6e9b1',
   						.
   						.
   	event: 'RespondToPickup',
    args: [Result] },
    					.
    					.
    event: 'PendingRequest',
       args: [Result] } ] }
```

[F9] The responses are received [ SGBNotifying() ] and the nearest vehicle amongst the nearby vehicles is calculated. [F10] The Optimal route for collecting the garbage from that SGB is sent to the nearest vehicle by calling the smart contract [ sendOptimalRoutePickUp() ] 

```javascript
truffle(ganache)> var nearestVehicle = nearbyVehicles[0]
undefined
truffle(ganache)> var optimalRoute = "GO TO LANE 4"
undefined
truffle(ganache)> vehicleManagement.sendOptimalRoutePickup(optimalRoute, nearestVehicle)
{ tx:
   '0xa11dd88167be72612ce9a5fd638536017832e32fe767a9d617e1be0de0439c5d',
   							.
   							.
   event: 'OptimalRoutePickup',
   args: [Result] } ] }
```

After receiving the optimal route, [F11] the nearest vehicle goes to the smart garbage bin to collect the waste [ wasteCollection() ]. Then, [F12] the pick Up information is recorded onto the blockchain by calling the smart contract [ setPickUpInformation() ].

Events Triggered: PickUpInformation, SGB_Update


```javascript
var pickUpInformation = "COLLECTED"
undefined
truffle(ganache)> vehicleManagement.setPickupInformation(SGB_ID, SGB_Loc, SGB_Status, nearestVehicle, pickUpInformation)
{ tx:
   '0x7abefc202cf87ddfda1c3d5e8c0febd8428f313afd68593926afce8ad44fa06d',
  receipt:
   { transactionHash:
      '0x7abefc202cf87ddfda1c3d5e8c0febd8428f313afd68593926afce8ad44fa06d',
      						.
      						.
      event: 'PickupInformation',
      args: [Result] } ] }
       						.
       						.
      event: 'SGB_Update',
      args: [Result] } ] }
```

Assumption: Vehicle is Full and ready to Dispose

[F13] The vehicle status of being full is recorded on to the blockchain by calling the smart contract [ setVehicleStatus() ]. [F14] The vehicle status is monitored [ monitorVehicleStatus() ] and since it is full, [F15] the optimal route for disposal is sent to the vehicle by calling the smart contract function [ sendOptimalRouteDisposal() ].

Events Triggered: VehicleStatus, SendOptimalRouteDisposal.
```javascript
truffle(ganache)> var vehicleStatus = "FULL"
undefined
truffle(ganache)> vehicleManagement.setVehicleStatus(nearestVehicle, vehicleStatus)
{ tx:
   '0x95e71c1cd3aa1ec2b0a54880e0471b771e66223d8cecff496e561fe950c0fb03',
   								.
   								.
   	event: 'VehicleStatus',
       args: [Result] } ] }
truffle(ganache)> vehicleManagement.sendOptimalRouteDisposal(optimalRoute, nearestVehicle)
{ tx:
   '0x983914c74ed7acc583e3fe5ab96c3a6cebc8a995fab968e324508a3d50d31508',
   								.
   								.
event: 'SendOptimalRouteDisposal',
       args: [Result] } ] }
```

Next, [F16] the vehicle disposes of the waste [ wasteDisposal() ] and [F17] records the disposal information on the blockchain by calling the smart contract function [ setDisposalInformation() ] which [F18] automatically sets the vehicleStatus to "AVAILABLE".

```javascript

truffle(ganache)> nearestVehicle = accounts[5]
'0xa488CcBB600be7f53B8b7fC1bd94F859ddf112B4'
truffle(ganache)> vehicleManagement.setDisposalInformation(nearestVehicle, disposalInformation)
{ tx:
   '0x12fb1077af8d9ea30c2db0b87a5c7be6fe7c2dfbde5173c997b25e4c9682be1a',
   							   .
   							   .
   	event: 'DisposalInformation',
    args: [Result] },
    						   .
    						   .
 	event: 'VehicleStatus',
    args: [Result] } ] }

```
						   
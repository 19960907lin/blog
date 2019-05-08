---
title: "Let's Understand the Technology behind the FairPlay DApp (with FAQs)"
date: 2019-05-07T11:30:23+08:00
draft: true
tags: ["technical","dapp"] 
categories: ["en"] 
---

This article will discolse some technical details of the FairPlay DApp. Also some frequent answers will be answered. If you haven't known this DApp, please have a look at [FairPlay: decentralized prize drawing for e-commerce](https://blog.cybermiles.io/post/20190502-fairplay-en/).

> Talk is cheap, show me your code!

Sure, please see [here](https://github.com/CyberMiles/smart_contracts/tree/master/FairPlay). Any pull request or issue is welocme ;)

## Overview

This DApp is implemented with normal Web technologies, using [web3-cmt.js](https://cybermiles.github.io/web3-cmt.js/) to interact with the smart contract. When you scan the QRcode (see [create the drawing](https://blog.cybermiles.io/post/20190502-fairplay-creator-en/) or [participate the drawing](https://blog.cybermiles.io/post/20190502-fairplay-player-en/)) in your CMT wallet, the DApp will be displayed with WebView objects. 

Any time the user creates a new FairPlay Drawing, a new contract will be deployed at a new address respectively. In the new contract, we use `mapping(address => Player) players;` to store the basic information of the participants. To be more specifically, the `Player` struct is as following:
``` 
struct Player {
	uint ts;
	string name;
	string contact;
	string mesg;
	string confirm_mesg;
}
```
`ts` indicates the time when the player participates the play. `name` and `contact` can be picked by the user as he wants. The player can enter `contact` in many ways, such as email, telegram, or wechat, etc. So it's worth mentioning that the user doesn't need to show his sensitive information, if he doesn't want. `mesg` is the note the player leaves when he participates, whereas the `confirm_mesg` is comment given by the winner.  Both two messages are public to each participant.


## FAQ
1. What will happen when different people in different time zones participate in the same FairPlay Drawing?
	* Interesting question. The answer is that the datetime shown in the DApp exactly varies from the user's local time zone. Surely, that also means it's impossible to deceive the contract about the time. Sounds really smart, right? The solution is as following:
		* **When create the play**: The datetime picked by the user as the end time, is firstly (1)converted to timestamp as seconds since unix epoch, according to the user's local time zone, then is (2)sent to the contract, where it is (3)compared with the special variable `now` (the current block timestamp as seconds since unix epoch), in order to make sure the drawing time is after now.
		* **When paticipate in the play**: The front-end (1)firstly get the timestamp from the contract, then (2)converts this timestamp to the readable datetime, according to the user's local time zone.
	
2. Is this DApp partially or completely decentralized? 
	* We are completely decentralized! Yeah, you might be wondering how we manage to save the pictures uploaded in a totally decentralized DApp, without a back-end server. It can be a possible way to encode the whole picture and then store it on our blockchain, theoretically. But that would be a horrible solution, with respect to the enormous size of the picture! Honestly, our DApp first directly uploads your picture from the browser to the [cloudinary platform](https://cloudinary.com/documentation/upload_images#direct_uploading_from_the_browser), a secure and public platform providing the media management service, and then fetch the url from this platform and store it on our blockchain.

3. Is the FairPlay DApp really fair?
	* Yes! We have built-in function [rand()](https://www.litylang.org/rand/) to generate the real random number. The relevant part of code to calculate the winners is as following:
		```
		// Construct the winner_addresses array
		for (i=0; i<player_addresses.length; i++) {
			cache.push(player_addresses[i]);
		}
		for (i=0; i<number_of_winners; i++) {
			uint r = rand() % cache.length;
			winner_addresses.push(cache[r]);
			cache[r] = cache[cache.length-1];
			delete cache[cache.length-1];
			cache.length--;
		}
		```

4. How many languages does it support now?
	* Now only the English and Chinese are supported. If you are interested in this DApp and willing to contribute the translation of other language, please go to follow the format of [this file](https://github.com/CyberMiles/smart_contracts/blob/master/FairPlay/dapp/js/language/en.js) and make a pull request. Thanks.

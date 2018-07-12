August 2018 challenge
=====================

This months challenge consists of you writing a bot to compete against other bots in the game of [COUP](http://gamegrumps.wikia.com/wiki/Coup).
We will have the first round of (1,000,000) games two weeks after the challenge has started.
Then another one a week after that and a last one a week after that.
The overall winner will be crowned and we have a discussion about the code we've written.

## RULEZ

1. Node only
1. No dependencies
1. No changes to engine
1. Name folder appropriately (so you can target specific bots)
1. No data sharing between games
1. No access to other bots
1. No changing other bots
1. No Internet
1. No js prototype changing
1. Your code has to stay inside your bots folder
1. Do not output to `stdout`
1. At the beginning of each round you add PRs to the repo (we only merge on the day the round begins)

## How to run the game?

The game comes with a simple "dumb" bot that just randomizes it's answers without checking much whether the actions are appropriate.
Each bot lives inside a folder and is named after that folder name.

```sh
.
├── bot1
│   └── index.js
├── bot2
│   └── index.js
├── bot3
│   └── index.js
│
├── README.md
├── constants.js
├── helper.js
├── index.js
└── test.js
```

To run the game `cd` into the challenge `201808` folder and run:

```sh
node index.js play
```

To run the test suit:

```sh
node test.js
```


## How do I build a bot?

- Create a folder in the root (next to the fake bot)
- Pick the name of the folder from the player list below
- Include an `index.js` file that exports below functions
- Run as many test rounds as you want to
- Create PR on the day of each round

You get to require 4 arrays from the engine at `index.js` inside your bot:

- `ALLPLAYER` An array of all players in the game `<Player>`
- `CARDS` An array of all 5 card types `<Card>`
- `DECK` An array of all cards in the deck (3 of each)
- `ACTIONS` An array of all actions `<Action>`

### `<Player>`

- `AbbasA`
- `BenC`
- `BorisB`
- `CharlesL`
- `JedW`
- `DomW`
- `JessT`
- `JohnM`
- `JossM`
- `KevinY`
- `LaurenA`
- `MalB`
- `MikeG`
- `MikeH`
- `NathS`
- `SanjiyaD`
- `TiciA`
- `TimL`
- `TomW`
- `TuanH`

### `<Card>`

- `duke`
- `assassin`
- `captain`
- `ambassador`
- `contessa`

### `<Action>`

- `taking-1`
- `foreign-aid`
- `couping`
- `taking-3`
- `assassination`
- `stealing`
- `swapping`

### `<CounterAction>`

- `foreign-aid` -> [`duke`, `false`],
- `assassination` -> [`contessa`, `false`],
- `stealing` -> [`captain`, `ambassador`, `false`],
- `taking-3` -> [`duke`, `false`],

### Functions to export

- `OnTurn`
	- Called when it is your turn to decide what you may want to do
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards }`
	- returns: `{ action: <Action>, against: <Player> }`
- `OnChallengeActionRound`
	- Called when another bot made an action and everyone get's to decide whether they want to challenge that action
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards, action, byWhom, toWhom }`
	- returns: `<Boolean>`
- `OnCounterAction`
	- Called when someone does something that can be countered with a card: `foreign-aid`, `stealing` and `assassination`
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards, action, byWhom }`
	- returns: `<CounterAction>`
- `OnCounterActionRound`
	- Called when a bot did a counter action and everyone get's to decided whether they want to challenge that counter action
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards, action, byWhom, toWhom }`
	- returns: `<Boolean>`
- `OnSwappingCards`
	- Called when you played your ambassador and now need to decide which cards you want to keep
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards, newCards }`
	- returns: `Array(<Card>)`
- `OnCardLoss`
	- Called when you lose a card to decide which one you want to lose
	- parameters: `{ history, myCards, myCoins, otherPlayers, discardedCards }`
	- returns: `<Card>`

### The parameters

Each function is passed one parameter object that can be deconstructed into the below items.

| parameter        | description                   |
|------------------|-------------------------------|
| `history`        | The history array. More below |
| `myCards`        | An array of your cards |
| `myCoins`        | The number of coins you have |
| `otherPlayers`   | An array of objects of each player, format: `[{ name: <Player>, coins: <Integer> }, { name: <Player>, coins: <Integer> }]` |
| `discardedCards` | An array of all cards that have been discarded so far (from penalties, coups or assassinations) |
| `action`         | The action that was taken `<Action>` |
| `byWhom`         | Who did the action `<Player>` |
| `toWhom`         | To whom is the action directed `<Player>` |
| `newCards`       | An array of cards for the ambassador swap `Array(<Card>)` |

### The history array

Each event is recorded in the history array. See below a list of all events and it's entires:

An action:
```
{
	type: 'action',
	action: <Action>,
	from: <Player>,
}
```

Lose a card:
```
{
	type: 'lost-card',
	player: <Player>,
	lost: <Card>,
}
```

Challenge outcome:
```
{
	type: 'challenge-round' || 'counter-round',
	challenger: <Player>,
	player: <Player>,
	action: <Action>,
	lying: <Boolean>,
}
```

A Penalty:
```
{
	type: 'penalty',
	from: <Player>,
}
```

An unsuccessful challenge:
```
{
	type: 'unsuccessful-challenge',
	action: 'swap-1',
	from: <Player>,
}
```

A counter action:
```
{
	type: 'counter-action',
	action: <Action>,
	from: <Player>,
	to: <Player>,
	counter: <Action>,
}
```

## How does the engine work?

The challenge algorithm:

```
if( taking-3, assassination, stealing, swapping )
	ChallengeRound via all bot.OnChallengeActionRound
		? false = continue
		: true = stop

if( foreign-aid, assassination, stealing )
	CounterAction via bot.OnCounterAction
		? false = continue
		: true = CounterChallengeRound via bot.OnCounterActionRound
			? false = continue
			: true = stop

else
	do-the-thing
```
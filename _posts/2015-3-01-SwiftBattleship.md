---
layout: post
title: iOS Swift Battleship Game
---
![_config.yml]({{ site.baseurl }}/images/battleship.jpg)

Here's an iOS battleship game in the Swift language. It's on github (link at bottom).
This is how to create the game using Swift.

To get started create a new project called "Battleship" in Xcode, using a single view application.

In your new project, create a new Swift file called Battle.swift and make sure you add it to the Battleship and BattleshipTests targets. Then, replace all the contents of the file with:

```
https://raw.githubusercontent.com/oliversarmy/Battleship/master/Battleship/Battle.swift
```

Check that your program compiles. The Battle class is the heart of the program. It's immutable. The functions the Battle class supports are (for Player1 or Player2):

* Place new ship at coordinates
* Randomly place all ships
* Shoot at coordinates

Navigate to the BattleshipTests group and create a new XCTestCase class called BattleshipAddShipTests.
Replace all the contents of the file with:

```
https://raw.githubusercontent.com/oliversarmy/Battleship/master/BattleshipTests/BattleshipAddShipTests.swift
```

Check the program compiles (remember Battle.swift is in the BattleshipTests target). Now run all the tests. The tests show we can add ships such that they are on the board and not on other ships.

The last test, testRandomBoard, checks that all ships are placed for both players. It also prints out a representation of the players boards. Open the debug/output pane at the bottom of Xcode and run testRandomBoard again. You can see where ships are placed for each player and if you run the test again the placement of the ships change. The ship is represented by an enum:

```swift
enum Ship: String, Printable  {
case Carrier = "A", Battleship = "B", Submarine = "S", Cruiser = "C", Patrol = "P"
//...
```

Create a new XCTestCase class called BattleshipBattleStateTests.
Replace all the contents of the file with:

```
https://raw.githubusercontent.com/oliversarmy/Battleship/master/BattleshipTests/BattleshipBattleStateTests.swift
```

Run the last test, testPlayer1Win. Sunk ships are displayed in lower case. Player1 has one patrol ship left, so she wins (Note: P and p look similar).

Now you have a minimally tested Battleship game. The testing is all done through the public API calls.

###Programming decisions

The sea is a multidimentional array of SeaScape. A Seascape is water, a miss or part of a ship, which may be damaged or sunk (all parts are sunk together).


```swift
// 2D board with ships, water and misses on it (Hits are on Ships!)
typealias BoardType = [[SeaScape]]

enum SeaScape: Printable {
    case Water, Miss
    case ShipSectionNominal(Ship)
    case ShipSectionDamaged(Ship)
    case ShipSectionSunk(Ship)
    var description: String {
        switch self {
        case .Water: return "_"
        case .Miss: return "~"
        case let .ShipSectionNominal(s): return s.description
        case .ShipSectionDamaged(_): return "X"
        case let .ShipSectionSunk(s): return s.description.lowercaseString
        }
    }
//...
```

Each API call results in a BattleOperation that contains the new Battle and a message indicating the operation's result. Within a method call, small amounts of mutable state may be created and I have chosen to highlight these by having each such variable start with "v" to indicate this.

```swift
// add a ship, but only for the setup stage and only if its not already added, is not on another ship or off the board
func addShip(ship: Ship, playerId: PlayerId, y: Int, x: Int, isVertical: Bool = false) -> BattleOperation {

    let board = battleStore.boardForPlayerId(playerId)
    switch battleStore.stateForPlayerId(playerId) {
    case .BoardSetupComplete: return BattleOperation(message:.AllShipsPlaced, battle: self)
    case .BoardSetup where BattleStore.isShip(ship, onBoard: board): return BattleOperation(message:.ShipAlreadyPlaced, battle: self)
    default: ()
    }

    if let pairs = BattleStore.pairsOverWaterForBoard(board, isVertical: isVertical, y: y, x: x, len: ship.shipLength) {
        var vBoard = board
        for (y, x) in pairs {
            vBoard[y][x] = SeaScape.ShipSectionNominal(ship)
        }

        return BattleOperation(message:.ShipPlaced, battle: newBattle(playerId, board: vBoard))
    } else {
        return BattleOperation(message:.ShipNotAllowedHere, battle: self)
    }
}
```

I pass a PlayerId to each API call. One might ask why not use a separate class for each player. Well you would end up with code like the following which is a bit of a hack:

```swift
// inner class doesnt have access to self. using a lazy var is one way round the limitation
private(set) lazy var player1: Player = Player(battle: self, idAndBoard: self.idAndBoard1, enemyIdAndBoard: self.idAndBoard2)
```

In places I chose to make static functions like pairsOverWaterForBoard (shown below). I originally had these as instance methods that used non mutable instance state. But, I found it cleaner, as well as easier to read and test to just make them functions.

```swift
// checks that the generated pairs are over water
private static func pairsOverWaterForBoard(board: BoardType, pairs: [(y:Int, x:Int)]) -> [(y:Int, x:Int)]? {
```

* [github/oliversarmy Battleship!] (https://github.com/oliversarmy/Battleship)


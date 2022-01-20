### prepend

```swift
import Foundation
import Combine

let numbers = (1...5).publisher
let publisher2 = (500...510).publisher

numbers
    .prepend(100, 101)
    .prepend([45, 67])
    .prepend(publisher2)
    .sink {
        print($0)
    }
// all values are combined...
// `pre`
```

### append

```swift
import Foundation
import Combine

let numbers = (1...5).publisher
let publisher2 = (500...510).publisher

numbers
    .append(11, 12)
    .append(13, 14)
    .append(publisher2)
    .sink {
        print($0)
    }
// 1,2,3,4,5,11,12,13,14,500,501....
```

### switchToLatest

```swift
import Foundation
import Combine

let publisher1 = PassthroughSubject<String, Never>()
let publisher2 = PassthroughSubject<String, Never>()

// parent publisher
let publishers = PassthroughSubject<PassthroughSubject<String, Never>, Never>()

publishers
    .switchToLatest()
    .sink {
        print($0)
    }

publishers.send(publisher1)

publisher1.send("Publisher 1 - Value 1")
publisher1.send("Publisher 1 - Value 2")

publishers.send(publisher2) // switching to publisher number 2

publisher2.send("Publisher 2 - Value 1")
publisher1.send("Publisher 1 - Value 3") // not going to happen, because publisher has switched to publisher number 2
```

### switchToLastest Practical Example

```swift
import UIKit
import Combine

let images = ["houston", "denver", "seattle"]
var index = 0

func getImage() -> AnyPublisher<UIImage?, Never> {
    
    return Future<UIImage?, Never> { promise in
        DispatchQueue.global().asyncAfter(deadline: .now() + 3.0) {
            promise(.success(UIImage(named: images[index])))
        }
    }
    .print()
    .map { $0 }
    .receive(on: RunLoop.main)
    .eraseToAnyPublisher()
    
}

let taps = PassthroughSubject<Void, Never>()

// if there's some lastest publisher, then it's going to switch to that publisher and then start reading
let subscription = taps.map { _ in getImage() }
    .print()
    .switchToLatest()
    .sink {
        print($0)
    }

taps.send()

DispatchQueue.main.asyncAfter(deadline: .now() + 6.0) {
    index += 1
    taps.send()
}

DispatchQueue.main.asyncAfter(deadline: .now() + 6.5) {
    index += 1
    taps.send()
}
```

### merge

```swift
import UIKit
import Combine

let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<Int, Never>()

publisher1.merge(with: publisher2).sink {
    print($0 )
}

publisher1.send(10)
publisher1.send(11)
publisher2.send(12)
publisher2.send(13)
```

### combineLastest

- combine them together in the form of a tuple

```swift
import UIKit
import Combine

let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

publisher1
    .combineLatest(publisher2)
    .sink {
        print("P1: \($0), P2: \($1)")
    }

publisher1.send(1) // 1
publisher2.send("A") // 1, A
publisher2.send("B") // 1, B 
```

### zip

- Try to receive a value from each of the publisher and then it simply allows them to combine together in the tuple

```swift
import UIKit
import Combine

let publisher1 = PassthroughSubject<Int, Never>()
let publisher2 = PassthroughSubject<String, Never>()

publisher1
    .zip(publisher2)
    .sink {
        print("P1: \($0), P2: \($1)")
    }

publisher1.send(1)
publisher1.send(2)
publisher2.send("3")
publisher2.send("4")

// p1: 1, p2: 3
// p1: 2, p2: 4
```

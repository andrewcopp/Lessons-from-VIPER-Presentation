
#[fit] Lessons from
#[fit] VIPER

^ I'm going to talk today about VIPER. I'm not here to preach the gospel of VIPER, although I might have if I had given this talk three months ago. Thankfully, I've had some time to reflect and my coworkers have called me out on things but there are some great lessons to be had that I want to share with you.

^ VIPER. I started learning about VIPER when I saw it come up on an Objc.io article. I thought it had a cool name. Started playing with it. Turns out it is an alternative to MVC.

---

# Agenda

* Refresher on MVC
* Lesson on VIPER
* Benefits
* Drawbacks

^ I will give a quick refresher on Model-View-Controller before jumping into the basics of VIPER. After that, I will talk about some of the strengths of VIPER as well as a few of the drawbacks.

---

![](https://docs.google.com/drawings/d/1pNYTDmtS572wiINoWlLDnQjNV_E1V_HkVjwCiagm4aY/pub?w=1440&h=1080)

^ Everyone here should know about MVC. It is one of the first things Apple teaches you about when learning to program for iOS. You have your views which appear on screen, your models which hold the data, and your controllers tie these two things together. But what about networking? Persistance? Serialization? Navigation? Animation? You may have answers to some of these but truthfully, we are all guility of doing some dumping in the ViewController.

---

#[fit] Massive
#[fit] View
#[fit] Controller

![](http://barfblog.com/wp-content/uploads/2013/12/pinoy-spaghetti-1024x683.jpg)

^ This leads to the other name for MVC, Massive View Controller. View controllers tend to bloat up as we fill them with extra things. This can be fondly referred to as spaghetti code. It is very tightly coupled.

---

![](https://docs.google.com/drawings/d/1AfQlkzsEi17sEoxLaxgZfWBMErykRbdAp_lU3Vs4L60/pub?w=1440&h=1080)

^ There are a few architectures that try to assist with this problems. Both MVP and MVVM acknowledge that the ViewController is so tightly coupled to the View that we should treat them as the same thing. The Presenter (in MVP) and the ViewModel (in MVVM) act as a midpoint to prep data from the models to be presented in the view controller. In my opinion, this doesn't go far enough.

---

# VIPER

![](http://7-themes.com/data_images/out/59/6972865-bush-viper-snake.jpg)

^ Enter VIPER. Viper stands for View, Interactor, Presenter, Entity, Router. VIPER addresses some of the holes with other architectures as it is an architecture for apps and not an architecture for features.

---

![](https://docs.google.com/drawings/d/19yvGBq53MQQpRBEaoDnw8xn99eAS8KBRXZTQj-OUlVY/pub?w=1440&h=1080)

^ Pictured is a basic layout of VIPER. You see that the View and ViewController are connected as you would expect. They are still responsible for configuring the view. The Presenter is in charge of formatting data into a viewable format. The Interactor handles the business logic of a feature. Entities are models that get handled by the Interactor. The Router is responsible for bringing all of this on screen and navigating to the next feature when necessary.

---

## Single Responsibility

^ VIPER does a few things really well. The first of which is it encourages you to adhere to the principle of single responsibility.

---

![inline](http://i.imgur.com/HQcLL4J.png)

^ In my experience, I've written many apps where ViewControllers and AppDelegates become classes that handle multiple things. As a result, they get pretty bloated. This graphs represents the number of lines in each file of basic todo app with an MVC architecture. Only a few classes but some of them are pretty darn big.

---

![inline](http://i.imgur.com/gVkkjnt.png)

^ This is the same app but using a VIPER architecture. A lot of files but they are all way smaller. The two biggest files are a category and a model for encapsulating data for a UITableViewDataSource. The two view controllers are not even bigger than 100 lines of code.

---

### Ravioli Code?

![](http://www.joondalupcatering.com.au/wp-content/uploads/2011/06/bigstock_Ravioli_1554931-562x373.jpg)

^ Is this ravioli code? I would argue not. Yes, there are a lot of classes but that doesn't mean we have poor cohesion. The cognitive load is small because we are operating inside of an agreed upon architecture.

---

### Circular Dependencies

^ At a recent iOSoho, I heard someone ask about circular dependencies and I thought, "I had that problem before VIPER!"

---

```swift
class NetworkManager {
  static let sharedInstance = NetworkManager()

  func downloadFiles() {
    NSURLSession.sharedSession().dataTaskWithURL(endpoint) { data, response, error in
      // Handle Errors
      // Serialize Response
      // Save Files
      DatabaseManager.sharedInstance.saveFiles(files)
    }
  }

  // Called by DatabaseManager
  func deleteFile(file: File) {
    // Serial File
    // Call NSURLSession
} }

class DatabaseManager {
  static let sharedInstance = DatabaseManager()

  // Called by Network Manager
  func saveFiles(files: [File]) {
    // Write to File
  }

  func deleteFile(file: File) {
    NetworkManager.sharedInstance.deleteFile() {
      do {
        try NSFileManager.defaultFileManager().removeItemAtPath(file.filePath)
        NetworkManager.sharedInstance().deleteFile(file)
      } catch { /* Handle Error */ }
      // Delete File using NSFileManager
      // Write to fi
} } }
```

^ I used to create managers for networking and persistence. I never was sure which of them I could create first because they both called upon the other one. As a result, I couldn't use dependency injection and had to lazy load singletons. This infected my code as I know had found an excuse to reuse these singletons through out my app.

---

```swift
class Network {
  func downloadFilesAt(endpoint: NSURL, completion: [File] -> Void) { ... }
  func deleteFile(file: File) { ... }
}

class Database {
  func saveFiles(files: [Files]) { ... }
  func deleteFile(file: [File], completion: File -> Void) { ... }
}

class ListInteractor {
  let network = Network() // Could be configured with dependency injection
  let database = Database() // Could be configured with dependency injection

  func downloadAndSaveFiles() {
    network.downloadFilesAt(endpoint) { files in
      self.database.saveFiles(files)
    }
  }

  func deleteFile(file: File) {
    database.deleteFile(file) { file in
      network.deleteFile(file)
    }
  }
}
```

^ VIPER taught me about moving the business logic outside of the managers. This logic belonged in its own class and freed me up to remove the singleton pattern from my apps.

---

## Program to Interface

^ Composition over inheritance. Another concept I had been hearing about but didn't know how to apply to my massive view controllers. By separating out all of the responsibilities, you can open up your app to a lot of modularization.

^ Reusable across projects. A/B testing. Unit testing.

^ Assuming you are using dependency injection and protocols.

^ Talk about mocking a network call. Talk about testing a view controller.

---

![](https://docs.google.com/drawings/d/19yvGBq53MQQpRBEaoDnw8xn99eAS8KBRXZTQj-OUlVY/pub?w=1440&h=1080)

^ Let's say you want to A/B test a new view. If your view and view controller are full of bloated code, that may be hard to do.

---

![](https://docs.google.com/drawings/d/1KzrYCJLm5B5fpO5nQscL0PiMH1xzArOxC74qcQUGByw/pub?w=1440&h=1080)

^ If your view and view controller are lightweight and hooked up to a presenter, they can be interchanged at runtime.

---

### Super helpful in testing

^ That might be a fairly trivial example. Where this really shines is in testing.

---

### How do I test asynchronous behavior?

^ I used to ask how can I test the asynchronous behavior of my app. You shouldn't. Well not directly. You shouldn't be going to the network in any case.

---

```swift
protocol NetworkProtocol {
  func downloadFiles(completion: [File] -> Void)
}

protocol DatabaseProtocol {
  func saveFiles(files: [File], completion: Bool -> Void)
}

func ListDataManager {
  let network: NetworkProtocol
  let database: DatabaseProtocol

  init(network: NetworkProtocol, database: DatabaseProtocol) {
    self.network = network
    self.database = database
  }

  func downloadAndSaveFiles(completion: Bool -> Void) {
    network.downloadFiles() { files in
      database.saveFiles(files) { completed in
        completion(completed)
      }
    }
  }
}
```

^ If you have set up all of your lean VIPER classes to conform to interfaces and you pass these interfaces in through dependency injection you are in business.

^ Some day this talk will include promises!

---

```swift
class MockNetwork: NetworkProtocol {
  func downloadFiles(completion: [File] -> Void)
    completion([] as! [File])
  }
}

class MockDatabase: DatabaseProtocol {
  func saveFiles(files: [File], completion: Bool -> Void)
    if files.count == 0 {
      completion(false)
    } else {
      completion(true)
    }
  }
}

func testThatUnsuccessfulFileDownloadDoesNotSave() {
  // GIVEN
  let listDataManager = listDataManager(network: MockNetwork(), database: MockDatabase())
  let expectation = expectationWithDescription("Method is asynchronous and cannot exit early")

  // WHEN
  listDataManager.downloadAndSaveFiles() { completed in
    // THEN
    XCTAssertFalse(completed)

    expectation.fulfil()
  }

  waitForExpectationsWithTimeout(1.0)
}
```

^ Create "mock" classes that conform to your protocols and pass them in like you would normal instances of a class. This gets the bahavior you want without having to go to the Internet or test Apple's frameworks.

---

### How do I test ViewControllers?

^ What about ViewControllers? Should you test them? No, they were written by Apple and don't need to be tested.

---

![](https://docs.google.com/drawings/d/1pNYTDmtS572wiINoWlLDnQjNV_E1V_HkVjwCiagm4aY/pub?w=1440&h=1080)

^ Imagine in your todo app you want to test that you can pull-to-refresh on your main feed.

---

![](https://docs.google.com/drawings/d/1uoZGf_XinlBD6EpUM20VgWZKhLRnsVatcR41P44zvO8/pub?w=1440&h=1080)

^ Normally, this would be hard because you would have to kickoff the process by finding a way to pull down on your table view and then verify the data is displayed correctly.

---

![](https://docs.google.com/drawings/d/19yvGBq53MQQpRBEaoDnw8xn99eAS8KBRXZTQj-OUlVY/pub?w=1440&h=1080)

^ You still aren't testing the view controller in VIPER but you can get pretty close. You should have a method in your presenter that is called when the view is pulled down.

---

![](https://docs.google.com/drawings/d/1P-6LZvOQM_zucWrWl1zHaFh5koirTu2R5os6kB76OwM/pub?w=1440&h=1080)

^ By calling this in your tests, you can verify what you want is actually happening. By creating these boundaries in your codebase, you can more easily divide things up and inspect the traffic across them.

---

## Organization

^ Some may not see this as much of a benefit as I do but I've always struggled to find a way of sorting a project that makes sense to me.

---

![fit](http://i.imgur.com/QPz7aQ1.png)

^ VIPER introduced me to the idea of organizing my code by classes that exist completely inside of modules and outside of modules. This separation of concerns would play nicely if I ever decided to spin off my service layer into a second app.

---

# Why VIPER can be dogmatic

---

## Boilerplate

* Feature
  * Wireframe
  * ViewController
  * Presenter
  * Interactor
  * DataManager

^ Now for some downsides. There is a fair amount of setup that goes into creating a new feature. When every feature requires a wireframe, view controller, presenter, interactor, and others on top of all the protocols that hook them together.

---

## Dumb Entities

```swift
struct TodoItem {
    let dueDate : NSDate
    let name : String

    init(dueDate: NSDate, name: String) {
        self.dueDate = dueDate
        self.name = name
    }

    // Formatting Methods
    ...

    // Transient Properties
    ...
}
```

^ VIPER recommends that your entities be nothing more than dumb data stores. I disagree with this. I think you should have as much code in here as possible. It's reusable and it keeps things in the value layer of your codebase.

---

## Instantiation

^ AppDependencies

```swift
class AppDependencies {
  func configureDependencies() {
    let coreDataStore = CoreDataStore()
    let clock = DeviceClock()
    let rootWireframe = RootWireframe()

    let listPresenter = ListPresenter()
    let listDataManager = ListDataManager()
    let listInteractor = ListInteractor(dataManager: listDataManager, clock: clock)

    let addWireframe = AddWireframe()
    let addInteractor = AddInteractor()
    let addPresenter = AddPresenter()
    let addDataManager = AddDataManager()

    // ...
  }
}
```

^ Instead of using factories, my experience with VIPER has been creating a configuration file that sets everything up when the app launches. This feels klunky and I don't want to keep everything in memory all the time. I've been experimenting with factories for creating modules but haven't found the time to perfect it yet.

---

# Conclusion

^ If you want to adopt VIPER in your organization, I think it goes a long way to helping nail down consistency. That being said, it doesn't offer anything that can't be accomplished in other ways. Single responsibility, programming to interfaces, etc. These can all be accomplished with whatever architecture you chose. However, VIPER was a way for me to start thinking outside the box and improve my coding abilities. I recommend everyone try it once to retrain their brain.

---

# Acknowledgements

* [Architecting iOS Apps with VIPER](https://www.objc.io/issues/13-architecture/viper/) - Jeff Gilbert and Conrad Stoll
* [Practically SOLID](https://speakerdeck.com/dannyhertz/practically-solid-1) - Danny Hertz
* [iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52#.myo5u9s2h) - Bohdan Orlov
* [Controlling Complexity in Swift](https://realm.io/news/andy-matuschak-controlling-complexity/) and [Let's Play: Refactor the Mega Controller!](https://realm.io/news/andy-matuschak-refactor-mega-controller/) - Andy Matuschak

^ Architecting iOS Apps with VIPER has a great repo attached. It's out of date but if you fix some basic Swift compiler errors it is a great learning tool.

---

# How's my public speaking?

^ Thanks for your time everyone. I'd love feedback on my public speaking and the content of my presentation. You can reach me on Twitter at @andrewcopp.

---

### We aren't hiring, but [Anchor](http://anchor.fm/jobs/ios) is!

^ Listen to bite-sized radio, then join the discussion. They are looking for a Lead iOS Engineer to come in and take over the Swift app.

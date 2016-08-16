# Flatiron Go

![](https://i.imgur.com/jWZDeRy.png)  

The various components of this app consist of using:
* [AVFoundation](https://developer.apple.com/av-foundation/)
* [CoreMotion](https://developer.apple.com/library/ios/documentation/CoreMotion/Reference/CoreMotion_Reference/)
* [UIKit](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIKit_Framework/)
* [Firebase](https://www.firebase.com)
* [GeoFire](https://github.com/firebase/geofire-objc)
* [Mapbox](https://www.mapbox.com/ios-sdk/)


# Firebase 
Setup a [Firebase](https://www.firebase.com) account. We will be utilizing the database & storage that a firebase account can provide. Get your XCode project up and running with firebase.

We installed Firebase into our Xcode project using Cocoapods. This isn't all the contents of our `Podfile`, just the portion that relates to Firebase.

```swift
pod 'Firebase'
pod 'Firebase/Database'
pod 'Firebase/Storage'
```

The [iOS guide](https://www.firebase.com/docs/ios/guide/) provided by firebase to setup your Xcode project is incredible. I highly recommend going through each step they provide in getting your project setup as opposed to winging it.

# Map

Open up your `Podfile`. 

Add the following lines to your `Podfile` underneath where it states **#Pods for ABC** (ABC being the name of your project).

```swift
pod 'GeoFire', :git => 'https://github.com/firebase/geofire-objc.git'
pod 'Mapbox-iOS-SDK', '~> 3.3'
```

We will be going into exactly what these two frameworks provide shortly. For now, I want to get your enviornment setup.

After you added that to your `Podfile`, go ahead and type in `pod install` within your Terminal. Make sure that you're within your directory when typing `pod install`, otherwise nothing will happen.

Lets create our `MapViewController.swift` file. This will be the entry point to our app. Hit File --> New  --> File then underneath the iOS section, highlight Source then highlight the Cocoa Touch Class option. Then click Next.

We want to make sure the Subclass of: option is set to `UIViewController`. The Language should be `Swift` and "Also create XIB file" should be unchecked. The Class: will be the name of our file so name it `MapViewController`. After doing so click Next. Like any new file you're creating, it should be associated with the Target you intend to write code for. In our case, we're interested in our Main Target (not any test target). So make sure that's checked (which should be by default), then click Create. 

Locate your `MapViewController.swift` file in the Project navigator. Remove the unnecessary code to where your file looks just like this (you can keep those comments at the top):

```swift
import UIKit

class MapViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
    }
    
}
```

Locate the `Main.storyboard` file in your project navigator. Locate that first View Controller Scene you see there. We need to select that scene and change the Custom Class of that View Controller to our `MapViewController`. Like so:

![](http://i.imgur.com/1nczYco.png?1)

Now head on back to the `MapViewController.swift` file, we're going to be setting up our various components programmatically.

Above the `viewDidLoad()` method, we will want to create an instance property. It will be a variable called `mapView` of type `MGLMapView!`. In order to use this `MGLMapView` type, you will first need to import `MapBox` at the top of the file here. If you don't see `Mapbox` auto-completing when you try to import it, hit command + b to compile your project. At this point, you should be able to import Mapbox.

```swift
var mapView: MGLMapView!
```

Within the `MapViewController.swift` file, below the last curly brace that encompasses our `MapViewController` class, we will want to create an extension on the `MapViewController` and create the following methods within that extension, like so:

```swift
// MARK: - Map View Methods
extension MapViewController {
    
    private func setupMapView() {
        mapView = MGLMapView(frame: view.bounds, styleURL: NSURL(string: "mapbox://styles/ianrahman/ciqodpgxe000681nm8xi1u1o9"))
        mapView.autoresizingMask = [.FlexibleWidth, .FlexibleHeight]
        mapView.delegate = self
        mapView.userTrackingMode = .Follow
        mapView.pitchEnabled = true
        
        view.addSubview(mapView)
        mapView.translatesAutoresizingMaskIntoConstraints = false
        mapView.leftAnchor.constraintEqualToAnchor(view.leftAnchor).active = true
        mapView.rightAnchor.constraintEqualToAnchor(view.rightAnchor).active = true
        mapView.bottomAnchor.constraintEqualToAnchor(view.bottomAnchor).active = true
        mapView.topAnchor.constraintEqualToAnchor(view.topAnchor).active = true
    }
    
}
```

We're creating this method here which will be available to any instance of the `MapViewController`. `setupMapView()`. The `setupMapView()` instance method initializes a `MGLMapView` and assigns this instance to our `mapView` property, that way we can utilize it throughout our application. We're adding it to our `view` and having it fit the entire screen.

Now head back to the `viewDidLoad()` method. We should add the following methods we just created to it.

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        setupMapView()
    }
```

Our `viewDidLoad()` function now does its thing by first calling on `super.viewDidLoad()` then proceeds to call on `setupMapView()` which creates that pretty map then adds it to our `view`.

# Users Current Location

Above `viewDidLoad()`, we need to add two more instance properties to our class. One will be a variable named `locationManager` which will be of type `CLLocationManager`. This is what it sounds like, it will act as our manager allowing us to see what the users current location is. The other instance property will be a variable called `userStartLocation` of type `CLLocation`. These two instance properties have been assigned a default value. We've used initializer syntax to provide default values to both of these instance properties.

```swift
var locationManager = CLLocationManager()
var userStartLocation = CLLocation()
```

Getting the users current location isn't that easy.

Below the extension we made which related to the Map View Methods, we will create another extension which deals solely with the users current location.

```swift
// MARK: - Current Location Methods
extension MapViewController: CLLocationManagerDelegate {
    
    func getUserLocation() -> CLLocation? {
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.requestWhenInUseAuthorization()
        locationManager.startMonitoringSignificantLocationChanges()
        
        let weHaveAuthorization = (CLLocationManager.authorizationStatus() == CLAuthorizationStatus.AuthorizedWhenInUse || CLLocationManager.authorizationStatus() == CLAuthorizationStatus.AuthorizedAlways)
        
        if weHaveAuthorization { return locationManager.location } else { return nil }
    }
    
    private func setupCurrentLocation() {
        if let location = getUserLocation() {
            userStartLocation = location
        }
    }
    
}
```

We've setup two functions. One called `getUserLocation()` which takes in no arguments but returns back a `CLLocation?`. The other called `setupCurrentLocation()` which will call on this `getUserLocation()` function and store the return value of `getUserLocation()` to the `userStartLocation` instance property if the value returned isn't nil.

Before we head back up to the `viewDidLoad()` function to add our new methods we made to it, there's one more thing we need to do.

Head over to the Map View Methods extension we made earlier. Below the `setupMapView()` function we created and implemented, add this method.

```swift
private func setCenterCoordinateOnMapView() {
        mapView.setCenterCoordinate(userStartLocation.coordinate, zoomLevel: 15, direction: 150, animated: false)
    }
```

When we call on this particular function, it will utilize the users current location and center the map on that particular point.

Heading back to the `viewDidLoad()` function, it should now look like this (after you add the necessary functions):

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        setupMapView()
        setupCurrentLocation()
        setCenterCoordinateOnMapView()
    }
```

Stepping through this, it will first call on `super.viewDidLoad()` then call on the functions we provided. It will setup the Map View, grab the users current location then center coordinate of the map on that users location.

# Treasures

Create a new `Treasure.swift` file with the following contents.

An instance of `Treasure` is made up of various properties and methods. It has a `location`, `name` and `item` which are most important. The `item` instance property will come into use later when we get to the Augmented Reality portion. The `Treasure` instance will be able to download its own image as well as create the `item` which is of type `CALayer` when it's required later. Any logic or functionality associated with a treasure object is done here in our implementation of the `Treasure` class to keeps things very simple for us.

```swift
import Foundation
import UIKit
import CoreLocation

final class Treasure: CustomStringConvertible {
    let location: GPSLocation
    var name: String
    var item = CALayer()
    var description: String { return makeDescription() }
    var image: UIImage?
    let imageURL: String
    var downloadingImage: Bool = false
    
    init(location: GPSLocation, name: String, imageURLString: String) {
        self.location = location
        self.name = name
        imageURL = imageURLString
    }
    
    func makeDescription() -> String {
        return "\(location.latitude), \(location.longitude)"
    }
    
    func createItem() {
        guard let image = image else {print("fix this later"); return }
        item.contents = image.CGImage
    }
    
    func makeImage(withCompletion completion: (Bool) -> ()) {
        guard downloadingImage == false else { print("Already downloading image."); return }
        guard image == nil else { print("We already have image"); return }
        guard let imageURL = NSURL(string: imageURL) else { print("Couldnt convert URL"); completion(false); return }
        
        downloadingImage = true
        
        let session = NSURLSession.sharedSession()
        
        session.dataTaskWithURL(imageURL) { [unowned self] data, response, error in
            dispatch_async(dispatch_get_main_queue(),{
                guard let data = data else { print("data came back nil"); completion(false); return }
                if let image = UIImage(data: data) { self.image = image }
                self.downloadingImage = false
                self.createItem()
                completion(true)
            })
            }.resume()
        
    }
}

struct GPSLocation {
    var latitude: Float
    var longitude: Float
}

struct TreasureLocation {
    var location: CLLocation
}
```

# Map (II)

Head back to the `MapViewController.swift` file. Lets create another extension, labeling it as "Treasure Methods"

```swift
// MARK: - Treasure Methods
extension MapViewController {
     
}
```

Our treasures are stored in Firebase as JSON. What does that mean?

![](http://i.imgur.com/tkIdqmr.png?1)

Well, we have a base URL which acts as our firebase reference. The name of this location is flatiron-go. At this location we have two other locations, one named `treasureLocations` and the other named `treasureProfiles`. Lets do some digging into the `treasureProfiles`.

![](http://i.imgur.com/NIm49XJ.png?1)

Lets look a little closer at one of these items. The first one.

![](http://i.imgur.com/taAmtKo.png?1)

The `kMrX9X2cUklCtYYjS9K` is a `key` within the `treasuresProfiles` location. With this `key`, we have access to another dictionary with two `key-value` pairs. One `key` is named `imageURL` where the value is  `String` representing a URL. The other `key` is called `name` and the value is a `String` representing the name of this treasure. Here, the name is "Hairy Harry".

The name is pretty self explanatory but the `imageURL` is where the actual image of Hairy Harry lives and where we can download it from within our app. That's perfect.

We don't want to store every single image of these various treasures in our iOS application. Imagine if we had 10,000 images or even 100,000 user-generated treasures, no-one would want to keep the app on their iPhone. Here, we're storing all of the information (including the images) of these treasure objects in Firebase. 

What about `treasureLocations`. What lives within that dictionary?

![](http://i.imgur.com/Drvv0Q6.png?1)

We can ignore the contents of this dictionary for now. What is important to take away from this right now is that the `key`'s here which look like a bunch of random numbers and letters match up with the `key`'s within the `treasureProfiles` dictionary. You might then ask, why have the locations and profile information of these treasure objects separate? How we're creating these treasure objects is something we will cover last here--but in working with GeoFire, we wanted to let it do its thing. Its thing being able to generate these values you see here (like 7zzzzzz) for the key g which can represent a specific lat & long on a map (which is awesome!). So we have a separate dictionary where the keys of these particular objects are unique where we can easily retrieve the name, image and location when we have a `key`. 

So now when we head back to Xcode, how can make the connection here to this particular URL which has all of this great info regarding treasures with our app?

*NOTE*: How did we get this info on firebase? That will be discussed shortly.

# Back to Xcode (maps stuff)

Going back to our extension, lets add a `typealias` in there to make our lives easier.

```swift
// MARK: - Treasure Methods
extension MapViewController {
    
    typealias ResponseDictionary = [String: AnyObject]
    
}
```

Now anytime we type out `ResponseDictionary`, it's the equivalent of typing out [`String`: `AnyObject`] which is a dictionary where the keys are `String`'s and the values are `AnyObject`. When dealing with the responses we will get back in our communications with Firebase, it takes this format - so we'll associate a word with it so that way we're not typing the same thing over and over again. We also get the benefit of Auto-Complete when typing out `ResponseDictionary` now.

Lets finally add a function to this extension now. We will call this function `setupGeoQueryWithLocation(_:)` It will take in one argument called `location` of type `CLLocation`. It will also return back a `GFCircleQuery`.

```swift
private func setupGeoQueryWithLocation(location: CLLocation) -> GFCircleQuery {
        let geofireRef = FIRDatabase.database().referenceWithPath(FIRReferencePath.treasureLocations)
        let geoFire = GeoFire(firebaseRef: geofireRef)
        let geoQuery = geoFire.queryAtLocation(location, withRadius: 10.0)
        return geoQuery
    }
``` 

In its implementation we're creating a constant named `geoFireRef`. That will equal (if you read that line of code) some path, but not just _any_ path. It will equal the location of the `treasureLocations` that we looked at earlier!

That next line of code will create a new constant called `geoFire`. We assign it a value which is an instance of `Geofire`. We initialize our `Geofire` object by passing in our `geofireRef` object to it, we get back an instance of `GeoFire` which has reference to the URL where our treasure locations live.

On the following line of code we are asking our `geoFire` object to create a query at the location that was passed into this function with a radius of 10.0. After we assign the return value of calling `queryAtLocation(_:withRadius:)` to a constant called `geoQuery`, we return `geoQuery` back to the caller of this function.

This is how we search for treasures that live within a certain radius of the location that is passed into this function.

To help us out (for a function we will soon write), lets create another function within this extension. This one will be called `generateLatAndLongFromLocation(_:)` that takes in one argument called `location` of type `CLLocation` and it will return a tuple of type (`Float`, `Float`).

```swift
private func generateLatAndLongFromLocation(location: CLLocation) -> (lat: Float, long: Float) {
        return (Float((location.coordinate.latitude)), Float((location.coordinate.longitude)))
    }
```

The implementation of this function is fairly simple. It returns back a tuple accessing the `location`'s properties (specifically the coordinate property and then the lat and long properties from that coordinate property). This is a helper function which we will use shortly.

Still within the same extension, lets create a function that will be able to create a `Treasure` object. We know what a `Treasure` object looks like, we designed it! We also know what a `Treasure` object looks like within Firebase. It's identified by that unique `key`. That unique `key` provides us access to the TreasureLocation and TreasureProfile (within Firebase). 

Lets think of what happens between our iOS app and Firebase as a conversation. We walk up to Firebase with our users current location. Firebase does its thing in locating treasure objects within the provided radius of the users current location. It hands us the unique `key` of the treasure object and its location (lat & long). It doesn't provide this to us in one big batch, for each one it finds that falls within that provided radius, it gives it to us. One at a time. Ok, so we're given a `key` and the location. We take that info and reach back up to a Firebase (for a second time), with the provided `key` and we ask it for the `name` and `imageURL` of the treasure object with this `key` we have. If you remember from earlier, the way we setup our database was to have the `TreasureLocations` split from the `TreasureProfiles`. This was because of how we have to work with GeoFire (in how we're creating these treasure objects within Xcode--which we will talk about). When we reach back up to Firebase that second time, what format is the response in? Meaning.. how is Firebase giving us back this name and imageURL? They're doing so as a dictionary.... drum roll... as that `ResponseDictionary` we created earlier. If you recall, the type of `ResponseDictionary` is [`String`: `AnyObject`]. 

With this info, lets create another function within this same extension called `saveTreasureLocally(response:key:location:)` that takes in three arguments. The first argument has an external name of `withResponse` and an internal name of `response` of type `ResponseDictionary`. The second argument is called `key` of type `String`. The third argument has an external name of `andLocation` and an internal name of `location` of type `GPSLocation`. It doesn't return anything.

```swift
private func saveTreasureLocally(withResponse response: ResponseDictionary, key: String, andLocation location: GPSLocation) {
        let name = response["name"] as? String ?? ""
        let imageURL = response["imageURL"] as? String ?? ""
        let treasure = Treasure(location: location, name: name, imageURLString: imageURL)
        let newTreasure = (key, treasure)
        treasures.append(newTreasure)
        treasure.makeImage { _ in }
    }
``` 

In our implementation, we will first create a constant called `name` and assign it a value utilizing the `response` argument we take in. `response` is a dictionary of type [`String`: `AnyObject`] which means if were to access a value within this dictionary we have to cast it using as, as? or as!. We will go with as? coupled with optional chaining to produce a default value if it turns up nil. Working with dictionaries in Swift, they're might not be a value at a particular key, the key might not even exist, so you get back an optional of the type that should be there. So we're utilizing optional chaining here which can be read as follows:

Look to grab the value for key `name` within the `response` dictionary. If it's not nil in that there's a value there, treat it as a `String` and assign that value to our constant called `name`. If the value for key `name` is nil, then assign the string literal "" to our constant `name` and move on to the next piece of code.

We do something similar with `imageURL` in the following line.

Then on the third line of code, we create a `treasure` constant which is a `Treasure` instance instantiated with our three most important items, the `location`, `name`, and `imageURL`.

What is this `treasures` variable we're utilizing here? This is a new property we're about to make on our `MapViewController` class. Scroll all the way back to the top of your `MapViewController` class and lets add another instance property.

```swift
var treasures: [(String, Treasure)] = []
```

This is a new instance property which is a variable called `treasures` of type [(`String`, `Treasure`)] It's an `Array` of tuples. We assign it a default value being an empty array of tuples. 

Scrolling back down to your implementation of `saveTreasureLocally(response:key:location)`, we will append to this `treasures` property we have a tuple with the `key` being the first part of the tuple, and the `treasure` object being the second part of the tuple.

`makeImage` is then called on our `treasure` instance here. We had implemented this function earlier--for now I will ask that you type it in, but we will discuss this further when we utilize it.

Lets now create a function which will communicate with Firebase and retrieve the profile information. The profile information of a treasure item consists of its name and imageURL. In order to make this connection to firebase we will need to the `key` of the treasure item. This function will be called `getTreasureProfileFor(_:completion:)` with no return type. It's first argument will be called `key` of type `String`, the second argument will be called `completion` of type `(Bool) -> ()`. This is the signature of a function and functions can be types as well! The type of this function is `(Bool) -> ()` which means it has one argument of type `Bool` and it returns nothing. So whoever calls on this `getTreasureProfileFor(_:completion:)` function is required to provide it with two arguments (and expect nothing in return). Those two arguments are a `String` and a function that takes in a `Bool` as an argument and returns nothing.


```swift
private func getTreasureProfileFor(key: String, completion: (Bool) -> ()) {
        let profileRef = FIRDatabase.database().referenceWithPath(FIRReferencePath.treasureProfiles + "/" + key)
        
        profileRef.observeEventType(FIRDataEventType.Value, withBlock: { [unowned self] snapshot in
            guard let profile = snapshot.value as? ResponseDictionary,
                treasureLocation = self.treasureLocations[snapshot.key]
                else { print("Unable to produce snapshot value or key"); completion(false); return }
            
            self.saveTreasureLocally(withResponse: profile, key: snapshot.key, andLocation: treasureLocation)
            completion(true)
            })
    }
```

Lets step through the implementation. When this function is called, we have two things handed to us - a `key` which is of type `String` and a `completion` constant which is of type `(Bool) -> ()`, it's a function we can call on (whenever we want) within the scope of this function.

First things first, we're creating our connection to firebase. 

```swift
let profileRef = FIRDatabase.database().referenceWithPath(FIRReferencePath.treasureProfiles + "/" + key)
```

This constant, called `profileRef` is a direct connection going through the inter-webs to Firebase locating our exact treasure (the provided `key`). With this constant, we're now able to call on a specific method available to the `FIRDatabaseReference` type which is the what the type of `profileRef` is.

Calling `observeEventType` on this constant, we provide it with a function where one of the arguments is `snapshot` of type `FIRDataSnapshot`. This argument, provided to us when `observeEventType` decides to call on this provided function contains the information we're looking for. 

Scrolling back up to above the `viewDidLoad()` function, we need to add another property:

```swift
var treasureLocations: [String: GPSLocation] = [:]
```

We will be adding items to this later. It will be a place that stores our loctations where the Keys to this dictionary will be the `key` `String` associated with the treasure and the value will be their location as a `GPSLocation` coordinate. The `GPSLocation` struct is a type we made within `Treasure.swift` file.

At this point, we would have already made the query up to firebase, making the request for treasures that fall within a certain radius and have stored them in this `treasureLocations` dictionary. So here, we have the `key` to our treasure-- we're going back up to firebase getting back a snapshot that now contains the name and imageURL within our `profile` constant. We're passing along this info to the `saveTreasureLocally` function we created earlier which will use this info to create a `Treasure` object and store it locally. After we do that, we will call on the `completion` argument handed to us in this function, passing in the value `true` to let the person who called on this function know that we are done!

This piece of code handed over to the `observeEventType` function on `profileRef` is happening asynchronously. It could take 2 seconds, it could take 5 min. but it's doing its thing as the rest of our app continues to run. It's not blocking the main thread, it's not holding anything else up.

Lets implement the final piece of this puzzle. We want to create a function that will do all of these various parts in one (to make our life very easy). We will call the function `getTreasuresFor(_:completion:)`. It will take in two arguments. The first argument is called `location` of type `CLLocation` and the second argument is called `completion` of type `(Bool) -> ()`. It will return nothing.

We call on our `setupGeoQueryWithLocation()` function, passing in the `location` argument we receive. This sets up our `geoQuery` object which allows us to communicate with firebase. We have a query object created that's able to retrieve the `key`'s of treasures that fall within a certain radius. So the `observeEventType` function we call on the `geoQuery` object is able to provide us with a `key` and `location` which we utilize to step through the various functions we created above to store these treasures locally on the phone that fall within a certain radius of the users current location. Like I stated earlier, this block of code provided to the `observeEventType` function here gets called repeatedly within the implementation of the `observeEventType` function when its able to located a `key` and `location` that falls within radius--it doesn't give us all the info within one chunk, we get it one at a time.


```swift
private func getTreasuresFor(location: CLLocation, completion: (Bool) -> ()) {
        let geoQuery = setupGeoQueryWithLocation(location)
    
        geoQuery.observeEventType(.KeyEntered) { [unowned self] key, location in
            guard let geoKey = key,
                geoLocation = location
                else { print("No Key and/or No Location"); completion(false); return }
            
            let treasureLocation = self.generateLatAndLongFromLocation(geoLocation)
            
            self.treasureLocations[geoKey] = (GPSLocation(latitude: treasureLocation.lat, longitude: treasureLocation.long))
            
            self.getTreasureProfileFor(geoKey) { [unowned self] result in
                if result { self.createAnnotations() }
                completion(result)
            }
        }
    }
```

Within our implementation, we're calling on a function `createAnnotations()` which is what places our treasure annotation on the map (for our user to be able to interact with).

```swift
// MARK: - Annotation Methods
extension MapViewController {
    
    private func createAnnotations() {
        guard let (_, treasure) = treasures.last else { print("No last treasure"); return }
        generateAnnotationWithTreasure(treasure)
    }
    
    private func generateAnnotationWithTreasure(treasure: Treasure) {
        let newAnnotation = MGLPointAnnotation()
        let lat = Double(treasure.location.latitude)
        let long = Double(treasure.location.longitude)
        newAnnotation.coordinate = CLLocationCoordinate2D(latitude: lat, longitude: long)
        newAnnotation.title = treasure.name
        mapView.addAnnotation(newAnnotation)
        
        let key = String(newAnnotation.coordinate.latitude) + String(newAnnotation.coordinate.longitude)
        annotations[key] = treasure
    }
    
}
```

We're utilizing another instance property here which should be created above the `viewDidLoad() ` function.

```swift
var annotations: [String: Treasure] = [:]
```

We want to know add this new function we made to our `viewDidLoad()` function.

```swift
getTreasuresFor(userStartLocation) { _ in }
```

Our `viewDidLoad()` should now look like this:

```swift
override func viewDidLoad() {
        super.viewDidLoad()
        setupMapView()
        setupCurrentLocation()
        setCenterCoordinateOnMapView()
        getTreasuresFor(userStartLocation) { _ in }
    }
```








<a href='https://learn.co/lessons/HowToFlatironGO' data-visibility='hidden'>View this lesson on Learn.co</a>

# Flatiron Go

![](https://i.imgur.com/iH0HDpt.png)

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

# Being the Map View Delegate. Creating the treasure images on screen.



```swift
// MARK: - MapView Delegate Methods
extension MapViewController: MGLMapViewDelegate {
    
    func mapView(mapView: MGLMapView, viewForAnnotation annotation: MGLAnnotation) -> MGLAnnotationView? {
        
        guard annotation is MGLPointAnnotation else { return nil }
        
        let reuseIdentifier = String(annotation.coordinate.longitude)
        
        var annotationView = mapView.dequeueReusableAnnotationViewWithIdentifier(reuseIdentifier) as? TreasureAnnotationView
        
        if annotationView == nil {
            annotationView = TreasureAnnotationView(reuseIdentifier: reuseIdentifier)
            annotationView!.frame = CGRectMake(0, 0, 100, 100)
            annotationView!.scalesWithViewingDistance = false
            annotationView!.enabled = true
            
            let imageView = UIImageView(image: UIImage(named: "treasure"))
            imageView.contentMode = .ScaleAspectFit
            imageView.translatesAutoresizingMaskIntoConstraints = false
            
            annotationView!.addSubview(imageView)
            imageView.topAnchor.constraintEqualToAnchor(annotationView?.topAnchor).active = true
            imageView.bottomAnchor.constraintEqualToAnchor(annotationView?.bottomAnchor).active = true
            imageView.leftAnchor.constraintEqualToAnchor(annotationView?.leftAnchor).active = true
            imageView.rightAnchor.constraintEqualToAnchor(annotationView?.rightAnchor).active = true
        }
        
        let key = String(annotation.coordinate.latitude) + String(annotation.coordinate.longitude)
        if let associatedTreasure = annotations[key] {
            annotationView?.treasure = associatedTreasure
        }
        
        return annotationView
    }
    
    func mapView(mapView: MGLMapView, didSelectAnnotationView annotationView: MGLAnnotationView) {
        handleTapOfAnnotationView(annotationView)
    }
    
    func mapView(mapView: MGLMapView, didSelectAnnotation annotation: MGLAnnotation) {
        // TODO: User is in radius of tapped treasure.
    }
    
    func mapView(mapView: MGLMapView, annotationCanShowCallout annotation: MGLAnnotation) -> Bool {
        return true
    }
    
    func mapViewDidFinishLoadingMap(mapView: MGLMapView) {
        let camera = MGLMapCamera(lookingAtCenterCoordinate: mapView.centerCoordinate, fromDistance: 200, pitch: 60, heading: 180)
        mapView.setCamera(camera, withDuration: 2, animationTimingFunction: CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseInEaseOut))
    }
    
}
```

# What happens when they tap a treasure icon on the map

```swift
// MARK: - Segue Method
extension MapViewController {
    
    private func handleTapOfAnnotationView(annotationView: MGLAnnotationView) {
        if let annotation = annotationView as? TreasureAnnotationView {
            
            // Providing a default treasure value to the annotations.treasure property if its nil (for w/e reason)
            if annotation.treasure == nil {
                annotation.treasure = Treasure(location: GPSLocation(latitude: 40.0, longitude: 40.0), name: "Charging Bull", imageURLString: Constants.bullImage)
                annotation.treasure.makeImage() { [unowned self] success in
                    self.performSegueWithIdentifier("TreasureSegue", sender: annotationView)
                }
            } else {
                performSegueWithIdentifier("TreasureSegue", sender: annotationView)
            }
        }
    }
    
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        guard segue.identifier == "TreasureSegue" else { return }
        guard let destVC = segue.destinationViewController as? ViewController else { return }
        
        if let annotation = sender as? TreasureAnnotationView {
            destVC.treasure = annotation.treasure
        }
    }
}
```

# Augmented Reality Portion


![OtherBear](https://i.imgur.com/wKJBOh1.png)

We are now in our `ViewController.swift` file. How did we get here? When a treasure icon is tapped from the `MapViewController`, we segue over to this `ViewController`. In the `prepareForSegue(_:sender:)` method on the `MapViewController`, we're able to get a hold of the instance of the `ViewController` through the `segue`'s `destinationViewController` property. This segue connection was made in our `Main.storyboard` file. In that `prepareForSegue(_:sender:)` function on the `MapViewController`, we assigned a value to the following instance property on our `ViewController` (which is the `.destinationViewController`):

```swift
var treasure: Treasure!
```

That way, within our `viewDidLoad()` method on the `ViewController`, we have full access to a `treasure` object. The one that was tapped on the `MapViewController`. With this `treasure` object, we can go through with the necessary steps to displaying on screen.

Recap: Our `ViewController` has an instance property called `treasure` of type `Treasure!`. It's an implicitly unwrapped optional, that way it has a default value of `nil` when our `ViewController` is loaded into memory because we don't have access to the `UIViewController`'s `init` function and we can't add a stored property to a class on a `UIViewController` without assigning it a default value. By making it an implicitly unwrapped optional, it has a default value of `nil`. We assign it a value in the `prepareForSegue(_:sender:)` method on our instance of `MapViewController` through the `segue` parameter which is of type `UIStoryboardSegue`. The `segue` object knows where we're going (because we set this up within our `Main.storyboard` file. Through that `segue` object, we get a hold of the instance of our `ViewController`. At that moment, we assign a value to the `treasure` instance property. What value? Well, it has to be of type `Treasure`--it's the `Treasure` instance that was tapped on the map. That's the value we assign to this instance property.

# Brief Overview
  
* `viewDidLoad()` - Here the view is loaded into memory, we are just changing the `backgroundColor` property on our `view` instance.
* `viewWillAppear(_:)` - The view is about to appear on screen so we're calling on a method called `setupMainComponents()` which will go through all the necessary steps to getting this AR component to work.
* `setupMainComponents()` - This function calls on the following functions to get everything setup:
	* `setupCaptureCameraDevice()` - Setting up the camera device to put the user in a mode as if they're going to take a picture.
	* `setupPreviewLayer()` - Setting up our image to be displayed on screen.
	* `setupMotionManager()` - Setting up an object that allows us to listen to changes in the device when the user moves the iPhone around.
	* `setupGestureRecognizer()` - Setting up an object that listens for any tap on the screen. We want to see if a user was able to tap the image on screen.
	* `setupDismissButton()` - Setting up a `UIButon` to appear on screen after a user is able to tap the image (this will allow the user to go back to the map).
	

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = UIColor.blackColor()
    }
    
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        setupMainComponents()
    }
    
    private func setupMainComponents() {
        setupCaptureCameraDevice()
        setupPreviewLayer()
        setupMotionManager()
        setupGestureRecognizer()
        setupDismissButton()
    }
```

Without discussing exactly what these various instance properties do yet, I would ask that you include them above the `viewDidLoad()` for now as they will be used throughout these various methods.

```swift
    let captureSession = AVCaptureSession()
    let motionManager = CMMotionManager()
    var previewLayer: AVCaptureVideoPreviewLayer!
    var treasure: Treasure!
    var foundImageView: UIImageView!
    var dismissButton: UIButton!
    
    var quaternionX: Double = 0.0 {
        didSet {
            if !foundTreasure { treasure.item.center.y = (CGFloat(quaternionX) * view.bounds.size.width - 180) * 4.0 }
        }
    }
    
    var quaternionY: Double = 0.0 {
        didSet {
            if !foundTreasure { treasure.item.center.x = (CGFloat(quaternionY) * view.bounds.size.height + 100) * 4.0 }
        }
    }
    
    var foundTreasure = false
```

# setupCaptureCameraDevice()

What we will be creating:

```swift
// MARK: - AVFoundation Methods
extension ViewController {
    
    private func setupCaptureCameraDevice() {
        let cameraDevice = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)
        let cameraDeviceInput = try? AVCaptureDeviceInput(device: cameraDevice)
        guard let camera = cameraDeviceInput where captureSession.canAddInput(camera) else { return }
        captureSession.addInput(camera)
        captureSession.startRunning()
    }
        
}
```

**1** - Create an extension on the `ViewController`, adding a `// MARK: - ` above the extension labeled AVFoundation Methods
  
**2** - Within this extension, create a function named `setupCaptureCameraDevice()`. This method will take in no arguments and return no values. In our implementation we want to do the following:  

Create a constant called `cameraDevice` and assign it the value `AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)`. This `defaultDeviceWithMediaType(_:)` type method on the `AVCaptureDevice` type, when passing in the `AVMediaTypeVideo` `String` constant, will return back to us the built in camera that is primarily used for capture and recording. We are storing this value in our `cameraDevice` constant.   


Create a constant called `cameraDeviceInput` and assign it the value `try? AVCaptureDeviceInput(device: cameraDevice)`. The initializer we're calling on `AVCaptureDeviceInput` can fail, which is why we use the `try?` keyword here. We're not handling any error this might throw (something we should look to do if we want to release this app). This `init` takes in an argument of type `AVCaptureDevice` which is the same type of `cameraDevice`, the constant we just made. This initializer will create an instance of `AVCaptureDeviceInput` which can be used to capture data from an `AVCaptureDevice` (which is our constant `cameraDevice)` in an `AVCaptureSession` (which is our `captureSession` instance property which we haven't talked about yet). 


Because our `cameraDeviceInput` might be nil in that we've tried initializing this object calling on an `init` function which can fail, we need to check to see that `cameraDeviceInput` is not nil. We do that here by using the `guard` statement. Not only do we want to make sure the `cameraDeviceInput` is not nil, we want to make sure we can add it as input to our `captureSession`. There's a method on our `captureSession` instance property which is of type `AVCaptureSession` which allows us to check that we can indeed add this `cameraDeviceInput`  as input. If `cameraDeviceInput` is not nil, we store its value in our local constant named `camera` , check to see that we can add it as input then carryon.


Next we want to call on the `addInput(_:)` method available to instances of `AVCaptureSession`. So we call on the `addInput(_:)` method on our `captureSession` instance property passing in the `camera` instance.


Lastly, we call `startRunning()` on our `captureSession` instance property. This begins the flow of data from inputs to outputs connected to our `AvCaptureSession` instance. We are not handling any errors that might come of this `startRunning()` method which is something we should look into if we were to release this app.


# setupPreviewLayer()

What we will be creating:

```swift
// MARK: - AVFoundation Methods
extension ViewController {
    
    private func setupCaptureCameraDevice() {
        let cameraDevice = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)
        let cameraDeviceInput = try? AVCaptureDeviceInput(device: cameraDevice)
        guard let camera = cameraDeviceInput where captureSession.canAddInput(camera) else { return }
        captureSession.addInput(camera)
        captureSession.startRunning()
    }
    
    private func setupPreviewLayer() {
        previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.bounds
        
        if treasure.image != nil {
            let height = treasure.image!.size.height
            let width = treasure.image!.size.height
            treasure.item.bounds = CGRectMake(100.0, 100.0, width, height)
            treasure.item.position = CGPointMake(view.bounds.size.height / 2, view.bounds.size.width / 2)
            previewLayer.addSublayer(treasure.item)
            view.layer.addSublayer(previewLayer)
        }
    }
    
}
```

We're not creating a new extension here. We're adding to the one we had created in the previous step. We're creating a new method we're adding below the `setupCaptureCameraDevice` method.

**1** - Create a function named `setupPreviewLayer()` which takes in no arguments and returns no values. In our implementation we want to do the following:

Above our `viewDidLoad()`, we have an instance property

```swift
var previewLayer: AVCaptureVideoPreviewLayer!
```

The `AVCaptureVideoPreviewLayer` is a sublcass of `CALayer`. It means we gain all the functionality available to us with a `CALayer`. We need to create an instance of this class with an instance of a capture session to be previewed on screen (which we have setup!). So we initialize our `previewLayer` instance property passing in the `captureSession` object to the the `AVCaptureVideoPreviewLayer` initializer.

Next you need to setup the frame of this `previewLayer` to equal the `bounds` of our `view`. This `previewLayer` will now fill the screen. We haven't though added it to the `view` yet where it would be visible.

Our `treasure` instance property has an `image` property on it which can be `nil`. If it's not `nil` in that we were able to create an instance of this `Treasure` object and grab down the image correctly from firebase then we will move forward. 

Our `Treasure` object has an instance method which we are utilizing here:

```swift
var item = CALayer()
```

Firebase gives us back `NSData` which represents our image. We then create a `UIImage` using this `NSData` provided to us by Firebase. With that `UIImage` instance now stored on our `Treasure` object, we need to convert that to a `CALayer` object. Why? Because we need to add this particular image to the `previewLayer` which is what is displayed on screen. It's the camera preview where you can move the iPhone around and take photos / videos (except we're not going provide any functionality that will allow anyone to take a photo or video). And we can't add a `UIImage` which is a `UIView` to a `CALayer`. We can add a `CALayer` to a `CALayer`. So the `item` property on any `Treasure` instance is of type `CALayer`.

That conversion is happening in this function within `Treasure`'s implementation (if you want to take a look):

```swift
    func createItem() {
        guard let image = image else {print("fix this later"); return }
        item.contents = image.CGImage
    }
```

This is why we're adding `treasure.item` to the `previewLayer` in the function `addSublayer(_:)`. After we do that, we need to now add the `previewLayer` object to our `view`'s `layer` property in the `addSublayer(_:)` method.





### **1** - Setup our AVCaptureSession & tell it to start running

The `captureSession` used here is initialized in the declaration of the property on the `ViewController`

```swift
let captureSession = AVCaptureSession()
```

The `cameraDeviceInput` is an instance of `AVCaptureDeviceInput`. Checking first that we can add the `cameraDeviceInput` to the session we then move forward by adding the `cameraDeviceInput` to the `captureSession`. That's a mouth full, and if you want to know more of what's going on here - I recommend option clicking the various types of these objects and reading through the documentation. 

In short, this setups our camera and tell it to begin running!
```swift
let cameraDevice = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)
let cameraDeviceInput = try? AVCaptureDeviceInput(device: cameraDevice)
guard let camera = cameraDeviceInput where captureSession.canAddInput(camera) else { return }
captureSession.addInput(cameraDeviceInput)
captureSession.startRunning() 
```
---

### **2** - Setup the Preview Layer

The `previewLayer` is a property on the `ViewController`. It's an instance of `AVCaptureVideoPreviewLayer`. 

![PreviewLayer](https://i.imgur.com/0k76NAV.png)

We are first initializing it, then settings it's frame to equal the view's bounds (it will fill the entire screen). If the treasure object we were handed from the previous MapViewController's image is not nil, then we will move forward!

The item property on `treasure` is of type `CALayer`. See the `Treasure.swift` file for how this is created. We position it so it's within view when the preview layer launches.

We then add this `previewLayer` to the `view`.

We should now see our treasure on screen!


```swift
previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
previewLayer.frame = view.bounds
        
if treasure.image != nil {
    let height = treasure.image!.size.height
    let width = treasure.image!.size.height
    treasure.item.bounds = CGRectMake(100.0, 100.0, width, height)
    treasure.item.position = CGPointMake(view.bounds.size.height / 2, view.bounds.size.width / 2)
    previewLayer.addSublayer(treasure.item)
    view.layer.addSublayer(previewLayer)
}
 ```
---


### **3** - Setup the Motion Manager

The `motionManager` is a property on the `ViewController` which is initialized within the declaration of the property like so:
```swift
let motionManager = CMMotionManager()
```

In the implementation of the block of code passed into the `startDeviceMotionUpdatesToQueue` function call, we have a `motion` object we can work with of type `CMDeviceMotion`.

```swift
if motionManager.deviceMotionAvailable && motionManager.accelerometerAvailable {
      motionManager.deviceMotionUpdateInterval = 2.0 / 60.0
      motionManager.startDeviceMotionUpdatesToQueue(NSOperationQueue.currentQueue()!) { [unowned self] motion, error in
                
          if error != nil { print("wtf. \(error)"); return }
          guard let motion = motion else { print("Couldn't unwrap motion"); return }
                
          self.quaternionX = motion.attitude.quaternion.x
          self.quaternionY = motion.attitude.quaternion.y
     }
}
```

![motion](http://i.imgur.com/n25qC0B.png)

This is getting called continuously as the iPhone is moving and there are *MANY* properties on this object to take advantage of. For now, we're utilizing the `quaternion.x` and `quaternion.y` values to update our `quaternionX` and `quaternionY` properties on our `ViewController`. 

This is being done, because we have `didSet` observers on theses properties which will in turn update our `treasure` object on screen (to make it appear as if it's moving with you in real time).

Here is what those `disSet` observers look like:

```swift
var quaternionX: Double = 0.0 {
    didSet {
        if !foundTreasure { treasure.item.center.y = (CGFloat(quaternionX) * view.bounds.size.width - 180) * 4.0 }
     }
}

var quaternionY: Double = 0.0 {
    didSet {
        if !foundTreasure { treasure.item.center.x = (CGFloat(quaternionY) * view.bounds.size.height + 100) * 4.0 }
     }
}
```

---

### **4** - Setting up our Gesture Recognizer

We want to know when a user taps on the screen.. simple enough!

```swift
let gestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(viewTapped))
gestureRecognizer.cancelsTouchesInView = false
view.addGestureRecognizer(gestureRecognizer)
```
When a tap comes in, the `viewTapped()` method is called. One of the arguments to this method calld `gesture` is of type `UITapGestureRecognizer`. Through this object, we are able to tell where the tap occurred within the `view` and check it against where the `treasure` object is to see if their tap is within range of where the item is.

```swift
func viewTapped(gesture: UITapGestureRecognizer) {
 	  let location = gesture.locationInView(view)
      // See the Xcode project for how this was implemented 
}
```

Below is the implementation of the various methods called when a tap occurs to check to see that it's within range of where our `treasure` object currently lives on screen.

```swift
func viewTapped(gesture: UITapGestureRecognizer) {
    let location = gesture.locationInView(view)
        
    let topLeftX = Int(treasure.item.origin.x)
    let topRightX = topLeftX + Int(treasure.item.width)
    let topLeftY = Int(treasure.item.origin.y)
    let bottomLeftY = topLeftY + Int(treasure.item.height)
        
    guard topLeftX < topRightX && topLeftX < bottomLeftY else { return }
        
    let xRange = topLeftX...topRightX
    let yRange = topLeftY...bottomLeftY
         
    checkForRange(xRange, yRange, withLocation: location)
}
```

```swift
private func checkForRange(xRange: Range<Int>, _ yRange: Range<Int>, withLocation location: CGPoint) {
    guard foundTreasure == false else { return }
        
    let tapIsInRange = xRange.contains(Int(location.x)) && yRange.contains(Int(location.y))
        
    if tapIsInRange {
            
        foundTreasure = true
        motionManager.stopDeviceMotionUpdates()
        captureSession.stopRunning()
            
        treasure.item.springToMiddle(withDuration: 1.5, damping: 9, inView: view)
        treasure.item.centerInView(view)
            
        previewLayer.fadeOutWithDuration(1.0)
            
        animateInTreasure()
        animateInDismissButton()
        displayNameOfTreasure()
        displayDiscoverLabel()
            
    }
}
```

---

Check out the Xcode project to see how the spring animations were created after tapping the treasure object on screen.

![bear](http://i.imgur.com/nAwylOw.png)



<a href='https://learn.co/lessons/HowToFlatironGO' data-visibility='hidden'>View this lesson on Learn.co</a>

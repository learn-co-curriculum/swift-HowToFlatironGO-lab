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



<a href='https://learn.co/lessons/HowToFlatironGO' data-visibility='hidden'>View this lesson on Learn.co</a>

---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - swift

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

search: true
---

# Introduction

This document specifies the API for using the SpatialCanvas service.

# Install

> In your Podfile, add the following sources

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/xmartlabs/cocoapods-specs.git'
```

> And then, add the following line to your targets of interest:

```
pod 'SpatialCanvas', '~> 0.3.1'
```

For now, the SDK only support Cocoapods.

# Usage

## Behaviour

> Initialize the SDK in your AppDelegate

```swift
import SpatialCanvas

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        SpatialCanvas.shared.initialize(environment: .staging) // or .production
        return true
    }

}
```

> In your ViewController:

```swift
import ARKit
import SpatialCanvas
import UKit

class ViewController: UIViewController {

    @IBOutlet var sceneView: ARSCNView!

    var spatialCanvasRoom: SpatialCanvasRoom?

    override func viewDidLoad() {
        super.viewDidLoad()

        SpatialCanvas.shared.delegate = self
        getRooms()
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = .horizontal
        SpatialCanvas.shared.run(scene: sceneView, with: configuration)
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        SpatialCanvas.shared.pause()
    }

    @IBAction func insertObject(_ sender: UIButton) {
        let position: float3 = [0,0,0]

        // add your object to the SpatialCanvasRoom, the SDK will notify this add operation by calling the SpatialCanvasRoomDelegate didAddObject method.
        spatialCanvasRoom?.addObject(position: position, completion: ResultValue.handle {
            debugPrint("Object persisted \($0)")
        })
    }

    func getRooms() {
        SpatialCanvas.shared.getNearRooms { [weak self] result in
            switch result {
            case let .success(rooms):
                self?.show(rooms: rooms)
            case let .error(error):
                // show error
            }
        }
    }

    func show(rooms: [SpatialCanvasRoomDescriptor]) {
        let alert = UIAlertController(title: "Select a room", message: nil, preferredStyle: .actionSheet)
        alert.addAction(UIAlertAction(title: "Create new", style: .default, handler: { [weak self] _ in
            self?.createNewRoom()
        }))
        rooms.forEach { s in
            alert.addAction(UIAlertAction(title: s.name, style: .default, handler: { [weak self] _ in
                self?.restoreRoom(id: s.id)
            }))
        }
        present(alert, animated: true, completion: nil)
    }

    func restoreRoom(id: String) {
        SpatialCanvas.shared.restoreRoom(id: id, completion: ResultValue.handle { [weak self] in
            self?.spatialCanvasRoom = $0
            $0.delegate = self
            $0.startScan()
        })
    }

    func createNewRoom() {
      let roomScan = RoomScan()

      // perform a room scan by
      roomScan.start()

      /* let the user scan the room */

      roomScan.stop()
      roomScan.finish()

      SpatialCanvas.shared.createRoom(name: "Office", room: roomScan) { [weak self] result in
          switch result {
          case let .success(r):
              self?.spatialCanvasRoom = r
              r.delegate = self
          case let .error(error):
              // show error
          }
      }
    }

}
```

> When setting you as the delegate of `SpatialCanvas.shared` instance, you will get notified about potential SDK errors and also you will receive all events from `ARSessionDelegate`.
`SpatialCanvasDelegate` inherits from the `ARSessionDelegate` protocol.

```swift
extension ViewController: SpatialCanvasDelegate {

    // Generic fallback
    func spatialCanvas(_ spatialCanvas: SpatialCanvas, didFailWith error: Error) {
        debugPrint("ERROR \(error.localizedDescription)")
    }

}
```

> Once you restore an already existent room and start the scanning process by calling `room.startScan()` you will get notified when the SDK finds objects or updates their position in the 3D space.

```swift
extension ViewController: SpatialCanvasRoomDelegate {

    // called when the SDK successfully adds an object after calling spatialCanvasRoom.addObject(...)
    func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didAddObject object: SpatialCanvasObject) {
        // add your object to your sceneView
    }

    // called when the SDK finds objects after restoring a room and calling spatialCanvasRoom.startScan()
    func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didFindObjects objects: [SpatialCanvasObject]) {
        objects.forEach {
           let o = // get geometry and properties for object with persisted id = $0.id
           o.position = $0.position
           // add o to your sceneView
        }
    }

    // the SDK may update the objects positions
    func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didUpdateObjects objects: [SpatialCanvasObject]) {
      objects.forEach {
         let o = // get node from sceneView with id = $0.id
         // update its position
         o.position = $0.position
      }
    }

    // intended to collect feedback from users
    func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didMatchRoomAndWantsFeedback callback: (UIViewController) -> Void) {
        callback(self)
    }

}
```

Behaviour of the SDK:

On first place, you need to query the available rooms by calling `SpatialCanvas.shared.getNearRooms` or `SpatialCanvas.shared.getAllRooms`. You may want to present a UI with the list of fetched rooms and let the user select one or create a new one if he wants.  

If you create a new room from scratch, after a room scanning process, you can start adding objects to it in order to persist them. On the other hand, if you restore an existent one you will start a scanning process until the SDK notifies you that it found your objects.


## Components

Main components of the SDK:

- SpatialCanvas.shared instance:  
  The main entry point for the SDK. You should initialize it when your application launches, configure your ViewController as its delegate and use it to run an ARKit session.
- SpatialCanvasDelegate:  
  Delegate that extends the ARSessionDelegate.
- RoomScan:  
  Object that allows you to perform a scan of the room when creating a new room.
- SpatialCanvasRoom:  
  Object that represents a room. Lets you add persistent objects.
- SpatialCanvasRoomDelegate:  
  Notifies you whenever the room finds already persisted objects along with their ids and positions.

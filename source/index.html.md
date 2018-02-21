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

SpatialCanvas is an iOS SDK meant to enhance your application by providing a persistent augmented reality experience.
This document specifies the main API for developers.

# Requirements

- iOS 11.3+
- ARKit 1.5+

# Install

> In your Podfile, add the following line to your targets of interest:

```ruby
pod 'SpatialCanvas', '~> 0.5'
```

> In your application Info.plist, add the following property:

```XML
<key>NSLocationWhenInUseUsageDescription</key>
<string>This application will use the location for Augmented Reality persistence.</string>
```

For now, the SDK only supports [CocoaPods](https://cocoapods.org/) as its install method.


# Usage

The SDK lets you create and restore `rooms` which have persisted objects associated.
In order to accomplish this, it relies on the existence of a rectangled `master anchor` object that
should not move and be unique within the real-world space (for instance, a frame in the wall).

In order to create a room the following steps are required:

1. Give the room a unique name.
2. Scan it.
3. Scan a master anchor object.

On the other hand, when restoring you need to:

1. Select the room you want to restore.
2. Scan the master anchor object of the restored room.

# API

## SpatialCanvas
  The main entry point for the SDK. It exposes functions to create and restore rooms and it's accessed through the `SpatialCanvas.shared` singleton instance.

```swift
  weak var delegate: SpatialCanvasDelegate?
```
> Delegate of SDK. It extends the ARSessionDelegate protocol.

```swift
  func initialize()
```
> Initialize the SDK. This should be done when the application launches.

```swift
  func run(sceneView: ARSCNView, with configuration: ARWorldTrackingConfiguration)
```  
> Instead of calling `sceneView.session.run(configuration)` as you would normally do in an ARKit based app, you should call this function.

```swift
  func pause()
```
> Instead of calling `sceneView.session.pause()` you should call this function.

```swift
  func createRoom(name: String, room: RoomScan, masterAnchor: MasterAnchorScan, completion: @escaping Completion<ResultValue<SpatialCanvasRoom>>)
```
> Creates a room with the given `name`, `room scan` and `masterAnchor`.

```swift
  func restoreRoom(id: String, completion: @escaping Completion<ResultValue<SpatialCanvasRoom>>)
```
> Restores a room with the given `id`.

```swift
  func getNearRooms(completion: @escaping Completion<ResultValue<[SpatialCanvasRoomDescriptor]>>)
```
> Retrieves a list of near rooms based on the current GPS location.

```swift
  func deleteRoom(roomId: String, completion: @escaping Completion<Result>)
```
> Deletes a room with the given id.  

## SpatialCanvasDelegate

Delegate that extends the `ARSessionDelegate`. You should set your view controller as the `delegate`
on the `SpatialCanvas.shared.delegate` variable.  
Instead of conforming to the `ARSessionDelegate`, you would conform to the `SpatialCanvasDelegate`.

```swift
extension ViewController: SpatialCanvasDelegate {

    // functions from ARSessionDelegate

    func spatialCanvas(_ spatialCanvas: SpatialCanvas, didFailWith error: Error) {
        // an SDK error has occurred
    }

}
```

## RoomScan

Lets you scan the room you are creating. An instance of this class is required to call
the `SpatialCanvas.shared.createRoom` function.

```swift
  func start()
```
> Starts the scan.

```swift
  func stop()
```
> Stops the scan. You can still re-start it from its current state.

```swift
  func finish()
```
> Call this function once you are done with the scan.

```swift
  func clear()
```
> Resets the current state of the scanner.

```swift
  var onNewFrame: (([float3]) -> Void)?
```
> Notifies you about new scanned feature points.

```swift
  var onProgress: ((Float) -> Void)?
```
> Progress indicator in the range [0,1].

## MasterAnchorScan

Lets you scan the master anchor object of the room. An instance of this class is required to call
the `SpatialCanvas.shared.createRoom` function.
All described functions for the `RoomScan` apply for this class as well.

```swift
  init(visualizer: ImageSearchVisualizer? = nil)
```
> Creates an instance with an associated visualizer. The visualizer will be notified about the scanner state.

```swift
  func scan()
```
> Attempts to extract the current rectangled detected shape. In case of success, it will be notified to the visualizer.

## ImageSearchVisualizer

Protocol used to provide feedback about the current master anchor scan state.

```swift
protocol ImageSearchVisualizer {

  // on-screen rectangle
  func on(rectangle: Rectangle2D)

  // real-world metrics
  func on(width: Double, height: Double)

  // after a successful masterAnchorScan.scan() call
  func on(image: ImageWithMetrics)

  func onReset()
  func onNoRectangle()

}
```

## SpatialCanvasRoom

Represents a created or restored room. It lets you add objects and notifies you whenever objects are added, found or updated.

```swift
  weak var delegate: SpatialCanvasRoomDelegate?
```
> Delegate of the room. It will be notified when an object is found, added or updated. Objects are found only when restoring a room.

```swift
  let masterAnchor: MasterAnchor
```
> Master anchor object. It has an imageUrl property describing the scanned anchor object.

```swift
  func addObject(position: float3, eulerAngles: float3, completion: Completion<Result>? = nil)
```
> Adds an object to the room in the given position and orientation. The object addition will be notified through the delegate.

## SpatialCanvasRoomDelegate

Notifies you whenever objects are added, found or updated.

```swift
protocol SpatialCanvasRoomDelegate: class {

  // objects were found after restoring a room and scanning the master anchor object
  func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didFindObjects objects: [SpatialCanvasObject])

  // object was added to the room
  func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didAddObject object: SpatialCanvasObject)

  // objects already added or found were updated in the room
  func spatialCanvasRoom(_ spatialCanvasRoom: SpatialCanvasRoom, didUpdateObjects objects: [SpatialCanvasObject])

}
```

## SpatialCanvasRoomDescriptor

Room descriptor.

```swift
class SpatialCanvasRoomDescriptor: NSObject {

    let id: String
    let name: String

}
```

## SpatialCanvasObject

Represents an object being managed by the SDK.

```swift
class SpatialCanvasObject {

    let id: String
    let position: float3

    // orientation
    let eulerAngles: float3

}
```

# Example

In order to see an example of usage please visit [here](https://github.com/spatialcanvas/spatialcanvas-ios).

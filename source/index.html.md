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

This document specifies the API for using the MetaCloud service.

# Install

> In your Podfile, add the following sources

```
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/xmartlabs/cocoapods-specs.git'
```

> And then, add the following line to your targets of interest:

```
pod 'MetaCloud', '~> 0.1'
```

For now, the SDK only support Cocoapods.

# Usage

## Behaviour

>> The ConnectionManager ip & port is a temporary configuration. It allow algorithms to run on the C++ Server.

> Initialize the SDK in your AppDelegate

```swift
import MetaCloud

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        MetaCloud.shared.initialize()
        ConnectionManager.shared.ipAndPort = (ip: "192.168.181.46", port: 8000) // Just for now
        return true
    }

}
```

> In your ViewController:

```swift
import ARKit
import MetaCloud
import UKit

class ViewController: UIViewController {

    @IBOutlet var sceneView: ARSCNView!

    var metaCloudSession: MetaCloudSession?

    override func viewDidLoad() {
        super.viewDidLoad()

        MetaCloud.shared.delegate = self
    }

    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        MetaCloud.shared.run(scene: sceneView)
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        sceneView.session.pause()
    }

    @IBAction func insertObject(_ sender: UIButton) {
        let position: float3 = [0,0,0]

        // add your object to your sceneView

        // add your object to the MetaCloudSession
        metaCloudSession?.addObject(position: position, completion: ResultValue.handle {
            debugPrint("Object persisted \($0)")
        })
    }

    // If you want to reset the MetaCloud SDK.
    @IBAction func resetButtonDidTouch(_ sender: UIButton) {
        metaCloudSession = nil
        MetaCloud.shared.reset()

    }

}
```

> When setting you as the delegate of `MetaCloud.shared` instance, you will get notified about the following events:

```swift
extension ViewController: MetaCloudDelegate {

    // Called when the SDK didn't find sessions associated with your GPS location.
    func metaCloud(_ metaCloud: MetaCloud, didDetectNewSession factory: SessionFactory) {
        debugPrint("New session detected")
        factory.createSession(name: "Xmartlabs", completion: ResultValue.handle { [weak self] in
            self?.metaCloudSession = $0
            $0.delegate = self
        })
    }

    // Called when the SDK found sessions associated with your GPS location.
    func metaCloud(_ metaCloud: MetaCloud, didFindExistentSessions sessions: [MetaCloudSessionDescriptor], factory: SessionFactory) {
        debugPrint("Existent sessions \(sessions)")
        // present to the user the possible sessions and then restore one of them.

        let selectedId = session[0].id
        factory.restoreSession(id: selectedId, completion: ResultValue.handle { [weak self] in
            self?.metaCloudSession = $0
            $0.delegate = self

            // once you are ready, start the scan process to find a match.
            $0.startScan()
        })
    }

    // Generic fallback
    func metaCloud(_ metaCloud: MetaCloud, didFailWith error: Error) {
        debugPrint("ERROR \(error.localizedDescription)")
    }

}
```

> Once you restore an already existent session and start the scanning process by calling `session.startScan()` you will get notified when the SDK finds objects or updates their position in the 3D space.

```swift
extension ViewController: MetaCloudSessionDelegate {

    func metaCloudSession(_ metaCloudSession: MetaCloudSession, didFindObjects objects: [MetaCloudObject]) {
        objects.forEach {
           let o = // get geometry and properties for object with persisted id = $0.id
           o.position = $0.position
           // add o to your sceneView
        }
    }

    func metaCloudSession(_ metaCloudSession: MetaCloudSession, didUpdateObjects: [MetaCloudObject]) {
      objects.forEach {
         let o = // get node from sceneView with id = $0.id
         // update its position
         o.position = $0.position
      }
    }

}
```

Behaviour of the SDK:

On first place, the SDK will query a set of sessions associated with your GPS location and let you know about this result. You may want to present a UI with the list of fetched sessions and let the user select one or create a new one if he wants.  

If you create a new session from scratch, after a room scanning process, you can start adding objects to it in order to persist them. On the other hand, if you restore an existent one you will start a scanning process until the SDK notifies you that it found your objects.

<aside class="notice">
The room scanning process is not included in this SDK version.
</aside>


## Components

Main components of the SDK:

- MetaCloud.shared instance:  
  The main entry point for the SDK. You should initialize it when your application launches, configure your ViewController as its delegate and use it to run an ARKit session.
- MetaCloudDelegate:  
  Delegate that notifies you about the sessions associated with your gps location.
- SessionFactory:  
  Lets you create a new session, restore an existent one and query all your meta cloud sessions.
- MetaCloudSession:  
  Object that represents a session. Lets you add persistent objects.
- MetaCloudSessionDelegate:  
  Notifies you whenever the session finds a match with a list of objects along with their ids and positions.

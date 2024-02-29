---
date: 2024-02-29 09:11
description: How to get WIfi information (SSID,BSID) in SwifUI
tags: Wifi
---


# How to get Wifi information(SSID,BSID) in SwifUI


On iOS itt is possible to get Wifi information from the device that is connected. In order to get this info we have to:

 
- on Xcode -> Sisgning & Capabilities we need to add the `Access WIFI information` entitlement
- we have to request Location permission (see [here](https://www.instantmirage.com/basics/location/) for a detailed explanation)


Once both conditions are met, we can use this code:
```

import SwiftUI

import Foundation
import SystemConfiguration.CaptiveNetwork
import NetworkExtension
import CoreLocation


@Observable
class NewLocationManager {
    var location: CLLocation? = nil
    
    private let locationManager = CLLocationManager()
    
    func requestUserAuthorization() async throws {
        locationManager.requestWhenInUseAuthorization()
    }
    
    func startCurrentLocationUpdates() async throws {
        for try await locationUpdate in CLLocationUpdate.liveUpdates() {
            guard let location = locationUpdate.location else { return }

            self.location = location
        }
    }
}



struct ContentView: View {
    
    
    @State var newlocationManager = NewLocationManager()
    @State private var wifiSsid = ""
    @State private var wifiBssid = ""
    var body: some View {
        VStack {
          
            Text("WiFi SSID:  \(wifiSsid)")
            Text("WiFi BSID: \(wifiBssid)")
            
            Button() {
                getWIFISSID()
               
            }label: {
                Text("get WiFi info")
            }
        }.task {
            await getWifi()
            try? await newlocationManager.requestUserAuthorization()
            try? await newlocationManager.startCurrentLocationUpdates()
            // remember that nothing will run here until the for try await loop finishes
         
            
        }
    }
    
    
    func getWifi() async {
        let w = await NEHotspotNetwork.fetchCurrent()
        
        guard let w else {return}
      
            wifiSsid = w.ssid
            wifiBssid = w.bssid
     
    }
    
    func getWIFISSID()  {
        NEHotspotNetwork.fetchCurrent(completionHandler: { (network) in
          
            if let network {
                
                print(network.description)
                let networkSSID = network.ssid
                wifiSsid = networkSSID
                wifiBssid = network.bssid
                print(networkSSID)
                print("Network: \(networkSSID) and signal strength %d",  network.signalStrength)
                

            } else {
                print("No available network")
            }
        })
    }
}



#Preview {
    ContentView()
}
```

And the Wifi info will be show on screen and in the logs:

```
<CNNetwork SSID VM7687600 BSSID 18:83:04:95:ba:ef [open] [signal 0] 0x28071c2d0>
VM7687600
Network:VM7687600 and signal strength %d 0.0
```





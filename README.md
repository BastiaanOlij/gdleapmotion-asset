# Godot Leap Motion GDNative module
This module is provided as is, all files are contained within the addons/g folder

The Orion/LeapC API used for this module has only been made available for Windows at this point in time. 
Linux/Mac versions are rumoured to become available and when this happens we will update this asset.

Source code for this module can be found here:
https://github.com/BastiaanOlij/gdleapmotion

While the leap motion can be used as a desktop pheripheral it really shines when used with a VR or AR headset. We are still working on supporting Project North Star.

Using Leap Motion in Godot
--------------------------
The leap motion driver has been implemented as a GDNative driver.

To use it, add a spatial node to your scene, name it something useful like "leap_motion" and then drag the file addons/gdleapmotion/gdlm_sensor.gdns onto the script property of this node.

You'll need to set the left hand and right hand scenes to scenes that need to be added when the leap motion starts tracking a hand. There are a couple of example scenes in the scenes subfolder of the add on.

Alternatively you can add ```Leap_Motion.tscn``` or ```Leap_Motion_with_collisions.tscn``` as a subscene to your project. These have preconfigured nodes ready for you.

Using Leap Motion in Godot with a VR headset
--------------------------------------------
Using the Leap Motion in VR takes an approach that is not immediately apparant. It would seem logical to add the Leap Motion node as a child node of the headset. While this will work you'll notice placement issues with the hands when you move your head.

Instead add your node as a child node of your ARVROrigin node. Then turn the ARVR property of the Leap Motion on. The driver will interact with the ARVR system to obtain the location and orientation of the HMD and perform the needed adjustments for hand placement as the sensor data on the leap motion is independently handled from the HMD.

Other properties of the driver
------------------------------
There are a few more properties that you can tweak.

The ```Smooth Factor``` allows you to set a smoothing factor for the tracking. If you experience a lot of jittering setting a lower value will smooth this out but at the price of increased lag.
Don't set this lower then 0.2. A value of 1.0 turns smoothing off.

The ```Hmd To Leap Motion``` transform only applies in ARVR mode and provides the transform with which the leap motion is placed in relation to the HMD.
Seeing I have an Oculus Rift and I am lazy, I've hardcoded the defaults for this to my Rift. If you're using a different HMD you will need to change this. You can do this as follows:
```
	# our default is 8cm from center between eyes to leap motion, you can change it here if you need to
	var hmd_to_leap = leap_motion.get_hmd_to_leap_motion()
	hmd_to_leap.origin = Vector3(0.0, 0.0, -0.08)
	leap_motion.set_hmd_to_leap_motion(hmd_to_leap)
```
Note that this is a full transform so if your leap motion is attached tilted you can include a rotation as well.

The ```Keep Hands For Frames``` setting tells the driver for how many frames you want to keep a hand "alive" after tracking is lost. Especially in VR where you can look away from your hands there can be nasty results when a hand just disappears.

The ```Keep Last Hand``` is an overrule for the previous setting. When turned on the driver will keep atleast one right hand and one left hand "alive" after tracking is lost.

Signals
-------
There are a number of signals that you can connect to on the GDNative module, you can do this as follows in your ```_ready``` function:
```
	leap_motion.connect("new_hand", self, "new_hand")
	leap_motion.connect("about_to_remove_hand", self, "about_to_remove_hand")
```

The ```new_hand``` signal is issued when a new subscene for a hand is instantiated and added. This gives you the opertunity to connect to any signals emitted from these scenes.

The ```about_to_remove_hand``` signal is issued right before the subscene for a hand is removed because tracking is lost.

Right now a subscene is added when the leap motion detects a hand, and removed when it looses tracking. In a future version we'll add tracking signals. 

Pinch and grab
--------------
Besides accurate orientation information the leap motion SDK also provides pinch and grab values that allow for more gesture based interactions. Most of the logic for this resided in our ```hand.gd``` which is the code associated with the subscenes we're adding.

The Leap Motion module will call set_pinch_distance, set_pinch_strength and set_grab_strength on the subscenes so these *must* be implemented.

```pinch_distance``` is the estimated distance between the top of your index finger and thumb.
```pinch_strength``` is the strength of the pinch, a value between 0.0 (finger tips are not touching) and 1.0 (fingers tips are touching)
```grab_strength``` is the strangth of a grab, a value between 0.0 (fist is open) and 1.0 (fist is closed)

When you look at the code in ```hand.gd``` you can see that we do not just make these values available, we also emit signals based on their value. Note specifically the ```pinched``` and ```grabbed``` signals that are implemented here.

Licensing
---------
The Godot Leap Motion module is supplied under an MIT License.
Please check www.leapmotion.com for the license for the LeapC library used to communicate with the Leap Motion device.

About this repository
---------------------
This repository was created by and is maintained by Bastiaan Olij a.k.a. Mux213

You can follow me on twitter for regular updates here:
https://twitter.com/mux213

Videos about my work with Godot including tutorials on working with the leap motion in Godot can by found on my youtube page:
https://www.youtube.com/channel/UCrbLJYzJjDf2p-vJC011lYw

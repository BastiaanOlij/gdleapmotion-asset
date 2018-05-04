# Godot Leap Motion GDNative module
This module is provided as is, all files are contained within the addons/g folder

The Orion/LeapC API used for this module has only been made available for Windows at this point in time. 
Linux/Mac versions are rumoured to become available and when this happens we will update this asset.

Source code for this module can be found here:
https://github.com/BastiaanOlij/gdleapmotion

While the leap motion can be used as a desktop pheripheral it really shines when used with a VR or AR headset. Support for using the leap motion with an AR or VR headset through the ARVR server is implemented in this module but requires Godot 3.0.3 or later.

Using Leap Motion in Godot
--------------------------
The leap motion driver has been implemented as a GDNative driver, in order to use it you need to place a few lines of code in the ```_ready``` function of a script preferably placed on your root node:

```
func _ready():
	# Create a new leap motion resource
	var leap_motion = preload("res://addons/gdleapmotion/gdlm_sensor.gdns").new()
	
	# Tell it what scenes to load when a new hand is detected
	leap_motion.set_left_hand_scene(preload("res://addons/gdleapmotion/scenes/left_hand_with_collisions.tscn"))
	leap_motion.set_right_hand_scene(preload("res://addons/gdleapmotion/scenes/right_hand_with_collisions.tscn"))
	
	# add this as a child node to our scene
	add_child(leap_motion)
```

The first line loads the ```gdlm_sensor.gdns``` file and creates a new node using that script.

The next two lines load scenes that will be instantiated when leap motion detects a new hand. You can create your own if you want the hands to look differently or add additional logic to them. Note that here versions with colliders are loaded.

Finally the last line adds the leap motion node as a child node to your current scene. It is important that the location of this node within your scene becomes the anchor point for the leap motion. Basically this point maps the physical location of your leap motion device to your virtual world.

Using Leap Motion in Godot with a VR headset
--------------------------------------------
Using the Leap Motion in VR takes an approach that is not immediately apparant. It would seem logical to instantiate the ```gdlm_sensor.gdns``` node as a child node of the headset. While this will work you'll notice placement issues with the hands when you move your head because the tracking of the headset and the tracking the leap motion performs are out of sync.

Instead create our node as a child node of your ARVROrigin node. Then turn the VR mode of the leap motion driver on. The driver will interact with the ARVR system to obtain the location and orientation of the HMD and perform the needed adjustments for hand placement as the sensor data on the leap motion is independently handled from the HMD.

In this case you would amend the ```_ready``` function to be something like:
```
func _ready():
	# enable vr interface for Oculus
	var interface = ARVRServer.find_interface("Oculus")
	if interface and interface.initialize():
		get_viewport().arvr = true
	
	# Create a new leap motion resource
	leap_motion = preload("res://addons/gdleapmotion/gdlm_sensor.gdns").new()

	# turn VR mode on
	leap_motion.set_arvr(true)
		
	# Tell it what scenes to load when a new hand is detected
	leap_motion.set_left_hand_scene(preload("res://addons/gdleapmotion/scenes/left_hand_with_collisions.tscn"))
	leap_motion.set_right_hand_scene(preload("res://addons/gdleapmotion/scenes/right_hand_with_collisions.tscn"))
	
	# Add it as a child to our origin. Our driver will pair it up with HMD automatically
	get_node("ARVROrigin").add_child(leap_motion)

```

I've added the initialisation of the VR interface here purely for example and they may be different for you if you use a different headset. The new bits are calling ```set_arvr``` and adding the node as a child node of our ARVROrigin node.

There are two more functions available you can add into the logic above:
```
	# you can change the smoothing we apply to the positioning though it seems to be ok even without. 1.0 is no smoothing. Don't set this lower then 0.2
	leap_motion.set_smooth_factor(0.5)
```
While the leap motion SDK does a really good job adjusting for the timing difference between two completely separate tracking system I did add some smoothing logic. This adds a small amount of interpolation to the positioning of the hands. I found that very little smoothing is needed, what I initially thoughts was an issue with the wild movement of your head was simply a result of small differences caused by the leap motion device not being perfectly centered on my HMD. 

I've left the option in place for now.

The second option is far more important. Your leap motion is attached to the front of the HMD and is therefor moved a certain distance forward. Seeing I have an Oculus Rift and I am lazy, I've hardcoded the defaults for this to my Rift. If you're using a different HMD you will need to change this as follows:
```
	# our default is 8cm from center between eyes to leap motion, you can change it here if you need to
	var hmd_to_leap = leap_motion.get_hmd_to_leap_motion()
	hmd_to_leap.origin = Vector3(0.0, 0.0, -0.08)
	leap_motion.set_hmd_to_leap_motion(hmd_to_leap)
```
Note that this is a full transform so if your leap motion is attached tilted you can include a rotation as well.

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

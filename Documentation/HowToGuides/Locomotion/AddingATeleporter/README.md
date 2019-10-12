&gt; [Home](../../../../README.md) &gt; [How-to Guides](../../README.md) &gt; [Locomotion](../README.md)

# Adding A Teleporter

> * Level: Beginner
>
> * Reading Time: 10 minutes
>
> * Checked with: Unity 2018.3.10f1

## Introduction

A popular way to move around a virtual space is via teleporting. This is the concept where the user can mark out a destination location within the virtual world and be transported to that location automatically without any further input.

There are two main types of teleporting:

* Instant - The user specifies a destination location (e.g.  marking it with a controller pointer) and then instantly appears at the destination usually with a camera fade to reduce motion sickness.
* Dash - The user specifies a destination location and the user is gradually moved over time in a linear motion until they reach their destination location.

## Useful definitions

* `Play Area` - The virtual space that contains a virtual player that represents the user's real world physically restricted environment.
* `Camera Blink` - To apply a fade color over the camera view and then fade away again to see the virtual world again, usually helps combat motion sickness.

## Prerequisites

* A Pointer exists in the scene. See either:
  * [Adding A Straight Pointer](../../Pointers/AddingAStraightPointer/README.md).
  * [Adding A Curved Pointer](../../Pointers/AddingACurvedPointer/README.md).

## Let's Start

### Step 1

Expand the VRTK directory in the Unity Project window until the `VRTK -> Prefabs -> Locomotion -> Teleporters` directory is visible to show the two current teleport choices of `Teleporter.Dash` and `Teleporter.Instant`. Both of these options work in a similar way and provide the same public API into the underlying prefab. For this example, we'll use the `Teleporter.Instant` prefab. Drag and drop the `Teleporter.Instant` prefab into the Hierarchy window.

![Drag Teleporter Instant To Hierarchy](assets/images/DragTeleporterInstantToHierarchy.png)

### Step 2

Select the `Teleporter.Instant` prefab in the Unity Hierarchy and change the `Teleporter Facade` component to configure the base functionality of the Teleporter.

We must specify some data so the teleporter knows what to move when we teleport and how to move it.

The `Target` parameter on the `Teleporter Facade` component determines which GameObject to actually move on the teleport method. Usually, the Camera Rig is the GameObject that we want to move as it contains our virtual player.

If the scene is set up with multiple SDK Camera Rigs due to wanting to support different hardware requirements, then you'll see it's not possible to assign all of these Camera Rigs to the `Target` parameter on the `Teleporter Facade` component. We also don't want to add multiple `Teleporter.Instant` prefabs to the scene for each Camera Rig.

This is where the `TrackedAlias` prefab helps out. The `TrackAlias` prefab provides aliases for the common virtual player GameObjects such as the Play Area, Headset and Controllers. See [Adding A TrackedAlias](../../CameraRigs/AddingATrackedAlias/README.md) for more information.

Expand the `TrackedAlias` GameObject in the Unity Hierarchy to expose the child GameObjects then drag and drop the `TrackedAlias -> PlayAreaAlias` GameObject into the `Target` parameter on the `TeleporterFacade` component.

This tells the Teleporter that when we teleport, we want to move the Play Area to the new location which will essentially move our virtual player to the new location.

> Note: Don't set the Headset as the `Target` because this is always updated to match the real world physical Headset position so the virtual GameObject position will always be instantly override.

![Drag And Drop Play Area Alias To Target Parameter](assets/images/DragAndDropPlayAreaAliasToTargetParameter.png)

### Step 3

When the `Target` is teleported, the destination location will become the new central position of the `Target` GameObject. Because our virtual Play Area represents our real world Play Area, this means that if the user walks around their real world Play Area then their position is actually offset inside the Play Area.

This can have an effect when teleporting the user because the user may actually expect to end up standing exactly on the spot where they marked as the destination location. If we just move the Play Area GameObject to match the destination location then the user will still be offset from that destination location and not end up where they expected.

![Diagram Of How Teleport Location May Require An Offset](assets/images/DiagramOfHowTeleportLocationMayRequireAnOffset.png)

This diagram shows the issue in where once the user teleports from their original location, because they are not standing in the center of their Play Area then after teleporting the Play Area matches the destination location but their actual location is not exactly on the spot where they marked as the destination location.

The user may actually expect when they teleport to be standing in the exact location of the destination location and therefore their current real world position in relation to their real world Play Area needs to be taken into consideration when moving the `Target`.

The `Offset` parameter on the `TeleporterFacade` component helps with this situation. The `Offset` parameter determines another GameObject that can be used to describe this offset to take into consideration when moving the `Target`, in this case the offset is the difference between the user's Headset position in relation to their Play Area.

So to provide this offset data to the Teleporter we simply need to drag and drop the `TrackAlias -> HeadsetAlias` GameObject into the `Offset` parameter on the `TeleporterFacade` component.

![Drag And Drop Headset Alias To Offset Parameter](assets/images/DragAndDropHeadsetAliasToOffsetParameter.png)

### Step 4

Both the `Teleporter.Instant` and `Teleporter.Dash` have the ability to fade the screen in and out by doing a Camera Blink whenever an instant location change occurs. This instant location change occurs on moving to the destination location for the `Teleporter.Instant` prefab and in both prefabs when the Teleporter snaps to the nearest floor to prevent the user walking out into space above a valid floor.

To perform this Camera Blink, the Teleporter needs to know about the cameras in the scene to apply the fade to. The `Camera Validity` parameter on the `TeleporterFacade` component allows us to specify a simple way of linking this up. The `TrackedAlias` prefab contains a `Rule` of all known scene cameras on the `TrackedAlias -> SceneCameras` GameObject so drag and drop the `TrackAlias -> SceneCameras` GameObject into the `Camera Validity` parameter on the `TeleporterFacade` component.

![Drag And Drop Scene Cameras To Camera Validity Parameter](assets/images/DragAndDropSceneCamerasToSceneCamerasParameter.png)

### Step 5

Now we have a fully working Teleporter in the scene, we just need a way to tell the Teleporter to move to the desired destination location.

We already have a Pointer prefab in our Unity Hierarchy so there is a way of marking out a destination location on valid areas of the virtual world. We just need to hook up the Pointer prefab to tell the Teleporter prefab to move to where it is pointing.

Let's set up the Pointer prefab in the scene so it will tell the Teleporter to teleport our user to the Pointer destination location when the Selection Action occurs.

The Pointer should already have an Activation Action of the Trackpad being touched will activate the Pointer, so let's make it so actually pressing the Trackpad down does the Selection Action for the Pointer.

This example assumes you are using the `Curved Pointer` prefab that was set up in the [Adding A Curved Pointer](../../Pointers/AddingACurvedPointer/README.md) guide.

Expand the `UnityXR.OpenVR.LeftController -> Trackpad` GameObject in the Unity Hierarchy to expose the `Press[8]` GameObject. This contains the Unity Button Action that listens for when the Left OpenVR Controller Trackpad button is pressed.

Select the `ObjectPointer.Curved` GameObject in the Unity Hierarchy then drag and drop the `UnityXR.OpenVR.LeftController -> Trackpad -> Press[8]` GameObject into the `Selection Action` parameter on the `Pointer Facade` component.

![Drag And Drop Trackpad Press to Selection Action Parameter](assets/images/DragAndDropTrackpadPresstoSelectionActionParameter.png)

### Step 6

The Curved Pointer prefab will now emit the destination location data whenever the user presses the Left OpenVR Controller Trackpad button so all we need to do is hook up that selection event to call the Teleporter.

Click the `+` symbol in the bottom right corner of the `Selected` event parameter on the `Pointer Facade` component found on the `ObjectPointer.Curved` GameObject then drag and drop the `Teleporter.Instant` GameObject into the box that appears and displays `None (Object)`.

![Drag And Drop Teleporter To Pointer Activated Event Listener](assets/images/DragAndDropTeleporterToPointerActivatedEventListener.png)

Select a `Function` to perform when the `Selected` event is emitted. For this example, select `TeleporterFacade -> Teleport` (be sure to select `Dynamic EventData - Teleport for this example).

![Set Selected Listener To Teleporter Teleport Action](assets/images/SetSelectedListenerToTeleporterTeleportAction.png)

### Done

Play the Unity scene and touch the Trackpad on the Left Controller and the Curved Pointer will emit the beam to the destination location on the floor where you wish to be teleported to. Pressing the Trackpad on the Left Controller will initiate the Selected event on the Curved Pointer, which in turn tells the Teleporter to teleport to the Pointer's destination. You will now be able to teleport around the scene.

## Related Reading

* Coming Soon...
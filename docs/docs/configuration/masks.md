---
id: masks
title: Masks
---

## Motion masks

Motion masks tell Frigate where motion should be ignored when deciding whether to run object detection or continue tracking an object.

### What motion masks do

- Motion detected inside a masked area is ignored when Frigate looks for activity worth analyzing.
- Motion in unmasked areas can start object detection and is used to update object tracking.
- The mask only affects the motion step. Once object detection is running, the detection region may still include parts of a masked area.

Typical places to use motion masks include:

- Timestamps or on-screen overlays
- Sky and clouds
- Rooftops
- Tree tops or distant foliage
- Roads or parking areas where you never care about motion

### What motion masks do not do

- They do not prevent object detection within the masked region once detection has started.
- They do not determine when clips, snapshots, or notifications are generated. Zones and `required_zones` should be used for event control.

### Over-masking and tracking quality

Frigate tracks objects over multiple frames. Motion near the previous bounding box helps predict where the object will move next. If large regions are masked:

- Objects may disappear when they move into a masked area.
- When they re-enter an unmasked area, they may be treated as entirely new objects.
- Fewer frames are available to build a reliable classification, causing slower recognition or missed detections.

As a guideline:

- Mask areas where interesting objects will never be (sky, rooftops, distant roads).
- Do not mask approach paths where you rely on accurate tracking (driveways, private sidewalks, yards, porches).

### Performance considerations

Motion and object detection both consume compute resources. Leaving busy regions unmasked (for example, a main street with continuous traffic) can:

- Trigger motion constantly
- Generate excessive detection regions
- Increase inference latency or cause dropped frames on lower-powered hardware

To reduce load without harming detection quality:

1. Motion-mask busy areas that never contain relevant objects (streets, public sidewalks, distant parking lots).
2. If needed, tune:
   - `detect.fps`
   - The resolution of the detect stream
   - The set of object types you detect (for example, only `person`)

Avoid masking areas where you still want tracking. For those locations, use zones to restrict when events are actually saved.

### Zones vs motion masks

Use these tools together:

- Motion masks reduce unnecessary motion processing and improve performance.
- Zones and `required_zones` control when events, clips, and notifications are generated.

A common pattern is to leave a private sidewalk unmasked so Frigate can track an approaching person, and then create a required zone on the porch so events fire only when the person reaches your door.

## Object filter masks

Object filter masks tell Frigate where a detection of a specific object type should be discarded as a false positive. They operate on the final detection step rather than on motion.

How they work:

- After object detection, Frigate checks the bottom center of the bounding box for each detected object.
- If this point lies within the object filter mask for that object type, the detection is ignored.

Typical uses:

- For people: mask rooftops, treetops, walls, skylines—places a person cannot stand.
- For cars: mask everywhere except the road or driveway.
- For persistent hotspots: precisely mask a small region where a static feature is repeatedly misclassified.

Object filter masks are independent of motion masks: motion can still be detected in these areas, but the resulting object detections of the masked type will be filtered out.

Object filter masks can be used to filter out stubborn false positives in fixed locations. For example, the base of this tree may be frequently detected as a person. The following image shows an example of an object filter mask (shaded red area) over the location where the bottom center is typically located to filter out person detections in a precise location.

![object mask](/img/bottom-center-mask.jpg)

When creating object filter masks, keep them as small and precise as possible so that valid detections are not excluded.

## Using the mask creator

The mask and zone editor in the Web UI can be used to draw both motion masks and object filter masks interactively over a still image from your camera.

To create a poly mask:

1. Visit the Web UI
2. Click/tap the gear icon and open "Settings"
3. Select "Mask / zone editor"
4. At the top right, select the camera you wish to create a mask or zone for
5. Click the plus icon under the type of mask or zone you would like to create
6. Click on the camera's latest image to create the points for a masked area. Click the first point again to close the polygon.
7. When you've finished creating your mask, press Save.
8. Restart Frigate to apply your changes.

Your config file will be updated with the relative coordinates of the mask/zone:

```yaml
motion:
  mask: "0.000,0.427,0.002,0.000,0.999,0.000,0.999,0.781,0.885,0.456,0.700,0.424,0.701,0.311,0.507,0.294,0.453,0.347,0.451,0.400"
```

Multiple masks can be listed in your config.

```yaml
motion:
  mask:
    - 0.239,1.246,0.175,0.901,0.165,0.805,0.195,0.802
    - 0.000,0.427,0.002,0.000,0.999,0.000,0.999,0.781,0.885,0.456
```

Object filter masks will appear in the configuration under the corresponding object type, but they are created in exactly the same way in the editor.

---
layout: post
title: "OHIF Import Measurements"
description: "How to import measurements into OHIF Viewer using measurementService.addRawMeasurement"
category: POST
tags: [OHIF, DICOM, Cornerstone, Medical Imaging, Measurements]
---

> The most relevant question raised in OHIF issues: https://github.com/OHIF/Viewers/issues/1800
> Version 3.10.2

![](https://github.com/user-attachments/assets/3f522c1b-7bd1-4423-a5c0-8995760cc9d3)

<!--more-->

> **Figure** The overall limitation of the single route per mode.

**Forming Working Prototype**. We first extract **trackedMeasurements** and use it as a reference format of data we going to deal with when importing measurments: 
```js
const measurements = measurementService.getMeasurements();
```

<details>

<summary>
Click to expand reference format
</summary>

```json
{
    "uid": "11570db5-d2a8-4868-a6e7-6017bc230710",
    "SOPInstanceUID": "1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188095",
    "FrameOfReferenceUID": "1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188070",
    "points": [ 
        [ 55.84648437499999, 305.7054700285197, 4 ], [ 97.74101562499999, 7.416407528519642, 4 ], [ 176.22343749999993, 170.52578252851958, 4 ], [ -22.635937500000068, 142.59609502851967, 4 ]
    ],
    "textBox": {
        "hasMoved": false,
        "worldPosition": [ 97.74101562499999, 156.56093877851964, 4 ],
        "worldBoundingBox": {
            "topLeft": [ 139.63554687499993, 198.4554700285196, 4 ],
            "topRight": [ 308.889453125, 198.4554700285196, 4 ],
            "bottomLeft": [ 139.63554687499993, 298.66718750000007, 4 ],
            "bottomRight": [ 308.889453125, 298.66718750000007, 4 ]
        }
    },
    "isLocked": false,
    "isVisible": true,
    "metadata": {
        "toolName": "Bidirectional",
        "viewPlaneNormal": [ 0, 0, -1 ],
        "viewUp": [ 0, -1, 0 ],
        "FrameOfReferenceUID": "1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188070",
        "referencedImageId": "wadors:http://127.0.0.1/OrthancNginx/dicom-web/studies/1.2.86.76547135.7.115532.20160916104421/series/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188084/instances/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188095/frames/1",
        "cameraFocalPoint": [ 176, 176, 4 ],
        "sliceIndex": 4
    },
    "referenceSeriesUID": "1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188084",
    "referenceStudyUID": "1.2.86.76547135.7.115532.20160916104421",
    "referencedImageId": "wadors:http://127.0.0.1/OrthancNginx/dicom-web/studies/1.2.86.76547135.7.115532.20160916104421/series/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188084/instances/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188095/frames/1",
    "frameNumber": 1,
    "toolName": "Bidirectional",
    "displaySetInstanceUID": "1ab64ffd-230c-d0aa-6161-00fa510e31b8",
    "label": "",
    "displayText": {
        "primary": [ "L: 301 mm", "W: 201 mm" ],
        "secondary": [ "S: 0 I: 4" ]
    },
    "data": {
        "imageId:wadors:http://127.0.0.1/OrthancNginx/dicom-web/studies/1.2.86.76547135.7.115532.20160916104421/series/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188084/instances/1.2.826.0.1.3680043.8.274.1.1.8323329.421317.1748947541.188095/frames/1": {
            "length": 301.21672688578593,
            "width": 200.81115125719057,
            "unit": "mm",
            "widthUnit": "mm"
        }
    },
    "type": "value_type::shortAxisLongAxis",
    "source": {
        "uid": "53d521c7-de53-3c20-1134-83cd2f1b77d7",
        "name": "Cornerstone3DTools",
        "version": "0.1"
    },
    "modifiedTimestamp": 1773825233,
    "isDirty": true,
    "isSelected": true
}

```

</details>

Then we use data to form the following request to measurement service (`addRawMeasurement`).

> **Important**: We need to place this code after cornerstone is avaialble to display related content.  

Next crucial problem is to reshape `measurements` and distribute them across `addRawMesurements` input parameters.

* First, we need `source` that is aligned with the one in `measurements` by `guid`. 
* The same is for further mappings, i.e. `sourceMappings`. 

1. For that, `measurementService` provides related API:

    ```js
    const CORNERSTONE_3D_TOOLS_SOURCE_NAME = 'Cornerstone3DTools';
    const CORNERSTONE_3D_TOOLS_SOURCE_VERSION = '0.1';

    const source = measurementService.getSource(
        CORNERSTONE_3D_TOOLS_SOURCE_NAME,
        CORNERSTONE_3D_TOOLS_SOURCE_VERSION
    );

    const mappings = measurementService.getSourceMappings(
        CORNERSTONE_3D_TOOLS_SOURCE_NAME,
        CORNERSTONE_3D_TOOLS_SOURCE_VERSION
    );
    ```

Because data don't have `cachedStats`. Do we have to manually register. We can assign default values `{}`. And it works!

  - However it should not be empty, as in UI you won't see info about H/Width of the measurement.
  ```js
      const annotation = {
        data: {
            // Manually add.
            cachedStats: {}
            ...
        }
      }
  ```

  - Therefore we need to put data with key equals to `referencedImageId` at `cachedStats` as follows:
  ```js
  // That duplicates the metadata content but would need for cornerstone to display.
  // See later: requires added imageId.
  'wadors:http://127.0.0.1/OrthancNginx/dicom-web/studies/1.2.86.76547135.7.1323518.20210710080000/series/1.2.826.0.1.3680043.8.274.1.1.8323329.418900.1748947477.387488/instances/1.2.826.0.1.3680043.8.274.1.1.8323329.418900.1748947477.387555/frames/1': {
    length: 301.21672688578593,
    width: 200.81115125719057,
    unit: 'mm',
    widthUnit: 'mm',
  },
  ```

![](https://github.com/user-attachments/assets/3a0a919c-e6f8-400a-87a7-56c77ba6129b)

**Register annotation UID as well**
    * This could be done as:
    ```js
    const annotation = {
        // annotationUID: toolData.annotation.annotationUID,
        annotationUID: '11570db5-d2a8-4868-a6e7-6017bc230710',
    }
    ```


Next, is **Problem on cornerstone side with rendering Bidirectional** measurement.

  * By exploring code, this step involves instantiation of the measurement (copy creation). I belive this process not entirely smooth and causes futher misalignments at rendering stage resulted in exceptions.

    * Then you get an exception:

      ```
      Error: getTargetIdImage: targetId must start with "imageId:" or "volumeId:"
      ```

      because key in cached data should start with "imageId:" ...

    * For some reason, cornerstone attempt to draw it with circular tool.
      Eitherway the problem for now is that no points (for some reason) is available to render this content.

      When this call happens.
      ```
      const svgDrawingHelper = getSvgDrawingHelper(element);
      ```
        at `drawHandles.js`
      ```
      drawHandle(svgDrawingHelper, annotationUID, handleGroupUID, handle, options, i);
      ```

      `handle` is `None`.

      After further investigations I found that `activeHandleCanvasCoords` is `undefined`, which is not equals to `None` (`Tools/dist/esm/tools/annotation/BidirectionalTool.js`)
      
      Expects `activeHandleIndex` that at least should be null (not `undefined`)

      **The solution**: cornerstone expects `activeHandleIndex: null`, so we have to add so in `handles`.

      ```js
      const annotations = {
        handles: {
            // Important for handling with cornerstone for Bidirectional type.
            activeHandleIndex: null
        }
      }
      ```

And that's it 🥳

**The function:**

```js
function importMeasurement(measurementService, measurement) {
  const CORNERSTONE_3D_TOOLS_SOURCE_NAME = 'Cornerstone3DTools';
  const CORNERSTONE_3D_TOOLS_SOURCE_VERSION = '0.1';

  const source = measurementService.getSource(
    CORNERSTONE_3D_TOOLS_SOURCE_NAME,
    CORNERSTONE_3D_TOOLS_SOURCE_VERSION
  );

  const mappings = measurementService.getSourceMappings(
    CORNERSTONE_3D_TOOLS_SOURCE_NAME,
    CORNERSTONE_3D_TOOLS_SOURCE_VERSION
  );

  const annotationType = measurement.metadata.toolName;

  
  // TODO. Add label restoration.
  const annotation = {
    annotationUID: measurement.uid,
    metadata: measurement.metadata,
    data: {
      cachedStats: measurement.data,
      handles: {
        activeHandleIndex: null,
        points: measurement.points,
      },
    },
  };

  const matchingMapping = mappings.find(m => m.annotationType === annotationType);

  measurementService.addRawMeasurement(
    source,
    annotationType,
    { annotation },
    matchingMapping.toMeasurementSchema
  );
}
```

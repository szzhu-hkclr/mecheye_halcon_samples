# HALCON Samples

This repository contains the HALCON sample program for Mech-Eye Industrial 3D Camera.

For more detailed information on controlling Mech-Eye Industrial 3D Camera with HALCON, please refer to the [user manual](https://docs.mech-mind.net/2.0/en-GB/MechEye/Genicam/Halcon/Halcon.html).
## Prerequisites

1. [HALCON](https://www.mvtec.com/downloads) 20.11 or above has been installed on IPC.
2. Mech-Eye SDK 2.0.0 or above has been installed on IPC.
3. Camera firmware version is consistent with Mech-Eye SDK version. If not, use Mech-Eye Viewer to upgrade the camera firmware.
4. The IP addresses of the camera and IPC are in the same subnet. Use Mech-Eye Viewer to set the camera IP address.

## Run the HALCON Sample Program

1. Open the sample program in HDevelop.
2. Obtain all the available cameras by running the **info_framegrabber** operator (click the Step Over button or press the F6 key).
   > Note: All the obtained cameras can be viewed in the **DeviceInfos** variable in the **Control Variables** area of the **Variable Window**.
3. Select a camera to connect in either of the following ways:
    - Select a camera by its **unique_name**:
    1. In the **Control Variables** area, double-click **DeviceInfos** to display a list of all the available cameras.
    2. In the list, double-click the camera that you want to connect, and copy the camera name after **unique_name:**
    3. In the **Program Window**, locate the line containing the **tuple_regexp_select** operator, and replace **MechEye** in single quotation marks with the copied **unique_name**.
    - Select a camera by its index number:
    1. In the **Control Variables** area, double-click **DeviceInfos** to display a list of all the available cameras.
    2. Check the index number of the camera that you want to connect.
    3. In the **Program Window**, locate the line containing the **open_framegrabber** operator, and change the number in **MechEyeDeviceInfos[X]** accordingly.
4. Run the sample program by clicking the Run button or pressing the F5 key.
5. If the program is successfully executed, you can find the saved 2D image (**image2d.bmp** by default) and point cloud (**PointCloud.ply** by default) in the folder where the sample program is located.

> Note:
> - After executing the entire sample program, please click the Reset Program Execution button or press the F2 key to reset program execution. Otherwise, the camera cannot be connected by Mech-Eye Viewer.
> - When the point cloud is displayed in the **Canvas** window, please click the orange Continue button in this window to continue the program execution. Otherwise, the program is stuck in the **visualize_object_model_3d** operator and will not proceed.

## Explanation of the Sample Program

-  Connect to the camera:

    ```
    info_framegrabber ('GigEVision2', 'device', Info, DeviceInfos)    
    tuple_regexp_select(DeviceInfos, 'MechEye', MechEyeDeviceInfos)
    open_framegrabber ('GigEVision2', 1, 1, 0, 0, 0, 0, 'default', -1, 'default', -1, 'false', 'default', MechEyeDeviceInfos[0],   0, -1, AcqHandle)
    ```

- Perform image capturing:

    ```
    grab_image (Image2d, AcqHandle)
    grab_data(Image3d, Region, Contours, AcqHandle, ObjectModel3D)
    ```

- Obtain available parameters:

    ```
    get_framegrabber_param (AcqHandle, 'available_param_names', ParameterValues)
    ```

- Modify parameters: Parameters included in the sample program are 2D parameters, 3D parameters, ROI, depth range and point cloud settings.

  - 2D parameters: 

    ```
    * Switch to the 2D scanning mode.
    set_framegrabber_param (AcqHandle, 'DeviceScanType', 'Areascan')
    set_framegrabber_param (AcqHandle, 'AcquisitionMode', 'SingleFrame')

    get_framegrabber_param (AcqHandle, 'Width', Width)
    get_framegrabber_param (AcqHandle, 'Height', Height)
    get_framegrabber_param (AcqHandle, 'PixelFormat', PixeLFormat)

    * Set the 2D scanning parameters.
    set_framegrabber_param (AcqHandle, 'Scan2DExposureMode', 'Timed')
    set_framegrabber_param (AcqHandle, 'Scan2DExposureTime', 100)    
    ```
  
  - 3D parameters:

    ```
    * Switch to the 3D scanning mode:
    set_framegrabber_param (AcqHandle, 'DeviceScanType', 'Areascan3D')

    * Open the 3D object model generator.
    set_framegrabber_param (AcqHandle, 'create_objectmodel3d', 'enable')
    set_framegrabber_param (AcqHandle, 'add_objectmodel3d_overlay_attrib', 'enable')

    get_framegrabber_param (AcqHandle, 'Width', Width)
    get_framegrabber_param (AcqHandle, 'Height', Height)
    get_framegrabber_param (AcqHandle, 'PixelFormat', PixeLFormat)

    * Set the 3D scanning exposure time parameter.
    set_framegrabber_param (AcqHandle, 'Scan3DExposureCount', 1)
    set_framegrabber_param (AcqHandle, 'Scan3DExposureTime', 8)
    ```
  
  - Depth range:

    ```
    * Set the 3D scanning depth range parameter(unit: mm).
    set_framegrabber_param (AcqHandle, 'DepthLowerLimit', 1)
    set_framegrabber_param (AcqHandle, 'DepthUpperLimit', 3000)
    ```
  
  - Point cloud settings:

    ```
    * Set the 3D scanning point cloud filter parameter.
    set_framegrabber_param (AcqHandle, 'CloudOutlierFilterMode', 'Normal')
    set_framegrabber_param (AcqHandle, 'CloudSmoothMode', 'Normal')
    ```

- Save the obtained data:

  - Save 2D image:

    ```
    * Acquire the 2D image from the camera.
    grab_image (Image2d, AcqHandle)

    * Save the 2D image result in "Image2d.bmp"
    write_image( Image2d , 'bmp' , 0 , 'Image2d' )
    ```
  
  - Save point cloud:

    ```
    * Acquire the 3D data from the camera.
    grab_data(Image3d, Region, Contours, AcqHandle, ObjectModel3D)

    * Save the point cloud result in "PointCloud.ply"
    write_object_model_3d (ObjectModel3D, 'ply', 'PointCloud.ply', [], [])
    ```
- Disconnect from the camera:
    
    ```
    clear_object_model_3d (ObjectModel3D)
    close_framegrabber (AcqHandle)
    ```
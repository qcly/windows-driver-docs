---
title: Supporting Multiple Processors
description: Supporting Multiple Processors
ms.assetid: 906d6b31-a447-4a94-b1a5-cd3028722db7
keywords: ["user-mode display drivers WDK Windows Vista , multiple processors", "multiple processor support WDK display", "runtime-handled multiple-processor optimizations WDK display", "driver-handled multiple-processor optimizations WDK display", "multiple-processor optimizations WDK display"]
---

# Supporting Multiple Processors


User-mode display drivers on multiple-processor computers can let the Microsoft Direct3D runtime handle multiple-processor optimizations, or the drivers can perform their own multiple-processor optimizations.

### <span id="Runtime-Handled_Multiple-Processor_Optimizations"></span><span id="runtime-handled_multiple-processor_optimizations"></span><span id="RUNTIME-HANDLED_MULTIPLE-PROCESSOR_OPTIMIZATIONS"></span>Runtime-Handled Multiple-Processor Optimizations

The multiple-processor optimizations that are handled by Direct3D runtime are enabled only on drivers that support the [**LockAsync**](https://msdn.microsoft.com/library/windows/hardware/ff568214), [**UnlockAsync**](https://msdn.microsoft.com/library/windows/hardware/ff570105), and [**Rename**](https://msdn.microsoft.com/library/windows/hardware/ff569245) functions. These functions enable the multiple-processor optimizations to work well with applications that frequently lock dynamic resources. The *LockAsync* and *UnlockAsync* functions--along with the [**GetQueryData**](https://msdn.microsoft.com/library/windows/hardware/ff566803) function--must be reentrant on drivers that expose a DDI version of 0x0000000B or greater. The driver returns the DDI-version value in the **DriverVersion** member of the [**D3D10DDIARG\_OPENADAPTER**](https://msdn.microsoft.com/library/windows/hardware/ff541724) structure in a call to the driver's [**OpenAdapter**](https://msdn.microsoft.com/library/windows/hardware/ff568601) function. When the runtime calls a driver function in a reentrant manner, one thread can execute inside that function while another thread that references the same display device executes inside of another driver function.

The Direct3D runtime uses multiple-processor optimizations in some situations to offload work to a separate processor and improve computer performance. When multiple-processor optimizations are enabled, an additional software layer is added between the Direct3D runtime and the user-mode display driver. This software layer intercepts all calls that the Direct3D runtime would otherwise make to the user-mode display driver's functions.

Instead of calling the user-mode display driver directly, the software layer queues commands into batches that a worker thread asynchronously processes. However, the software layer cannot batch all calls that are made to the user-mode display driver's functions. In particular, the software layer cannot batch calls to functions that return information (for example, [**CreateResource**](https://msdn.microsoft.com/library/windows/hardware/ff540688)). When the software layer must call one of these types of driver functions, it flushes all queued commands through the worker thread, and then the software layer calls the driver function on the main application thread.

### <span id="Driver-Handled_Multiple-Processor_Optimizations"></span><span id="driver-handled_multiple-processor_optimizations"></span><span id="DRIVER-HANDLED_MULTIPLE-PROCESSOR_OPTIMIZATIONS"></span>Driver-Handled Multiple-Processor Optimizations

If a driver will perform its own multiple-processor optimizations, it must not implement [**LockAsync**](https://msdn.microsoft.com/library/windows/hardware/ff568214), [**UnlockAsync**](https://msdn.microsoft.com/library/windows/hardware/ff570105), and [**Rename**](https://msdn.microsoft.com/library/windows/hardware/ff569245) functions. In this situation, the driver must call the [**pfnSetAsyncCallbacksCb**](https://msdn.microsoft.com/library/windows/hardware/ff568924) function to notify the runtime whether the runtime will start or stop receiving calls to the runtime's callback functions from a worker thread.

If the driver performs its own multiple-processor optimizations, it should follow the same policy that the Direct3D runtime uses when it determines to enable multiple-processor optimizations. This policy enables fair sharing of system resources across all processes. In particular, the driver should disable multiple-processor optimizations in the following situations:

-   The application runs in windowed mode.

-   The computer contains only one processor (or processor core); the driver should disable optimizations on single-processor computers with hyper-threading.

-   The application requested that no multiple-processor optimizations be enabled, or the application uses software-vertex processing; this information is passed to the driver's [**CreateDevice**](https://msdn.microsoft.com/library/windows/hardware/ff540634) function.

If vendors want to enable multiple-processor optimizations in one of these situations, they should first contact Microsoft.

 

 

[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20[display\display]:%20Supporting%20Multiple%20Processors%20%20RELEASE:%20%282/10/2017%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")





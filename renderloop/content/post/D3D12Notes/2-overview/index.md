---
title: "Direct3D 12 Notes Entry #2: Let's Code the First Application" # Title of the blog post.
date: 2025-01-22T20:28:24+02:00 # Date of post creation.
description: "Learning Direct3D 12: Personal Notes #2: Let's Code the First Application" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
series: "Learning Direct3D 12: Personal Notes"
usePageBundles: true # Set to true to group assets like images in the same folder as this post.
featureImage: "" # Sets featured image on blog post.
featureImageAlt: '' # Alternative text for featured image.
featureImageCap: '' # Caption (optional).
thumbnail: "thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Rendering
tags:
  - Directx 12
  - D3D
  - Rendering
  - C++
# comment: false # Disable comment if false.
---

<script>
    document.addEventListener("DOMContentLoaded", function() {
        renderMathInElement(document.body, {
            delimiters: [
                {left: "$$", right: "$$", display: true},
                {left: "$", right: "$", display: false}
            ]
        });
    });
</script>

**In this second chapter, the focus is to implement a simple application using the components and the flow described in the previous one. You start creating a very minimal application: drawing a static 2D triangle.**

<br/>

# First D3D 12 Application: Drawing a Triangle

This simple task already requires a certain effort, particularly in setting up the entire pipeline. Additionally, the practical implementation introduces some aspects and details that were ignored in the previous lesson, such as window initialization.

The code for this entry can be downloaded from https://github.com/salvatorespoto/D3DNotes. Instructions for compiling the code are included in the source files. All programs in this series are compiled and linked from the command line using clang++. The use of an IDE is avoided to maintain focus on the code..

Before delving into the development, let's discuss some generic aspects of Windows and Direct3D programming.

## Some General Concepts in Direct3D Programming

### COM (Component Object Model)

All the Direct3D objects we create and use expose their functionality through COM interfaces. These interfaces are wrapped in `ComPtr` template objects, which manage their lifetimes. `ComPtr` objects are smart pointers that automatically handle reference counting, ensuring that the interfaces are properly deallocated when no longer needed, thus preventing memory leaks.
As an example, the following code shows how a command queue is created.

```cpp
// Command queue interface is held in a ComPtr<> smart pointer
ComPtr<ID3D12CommandQueue> commandQueue;

// Fill the properties of the command queue we want to create
D3D12_COMMAND_QUEUE_DESC commandQueueDescriptor = {};
//
// ... populate the commandQueueDescriptor struct
//

// Create the command queue
d3dDevice->CreateCommandQueue(&commandQueueDescriptor, IID_PPV_ARGS(&commandQueue));
```

In this example, `ComPtr<ID3D12CommandQueue>` is used to hold the command queue COM interface. 

Something we’ll use often when creating objects is the macro **IID_PPV_ARGS**, that expands to two function arguments: the **RIID** (Reference Identifier) of the interface we want to create, which is a **unique identifier** for that interface, and the address of a pointer that will hold the new interface.

Using this macro is not mandatory, but it is a common practice because it helps prevent errors. According to the Windows documentation, a common mistake is passing an **RIID** that does not match the type of the interface being created. **IID_PPV_ARGS** generates these arguments automatically, ensuring the correct **RIID** and pointer type are used, thus avoiding such mistakes.

### **Microsoft DirectX Graphics Infrastructure** DXGI Factory

**Microsoft DirectX Graphics Infrastructure (DXGI)** is a component that provides several low-level functionalities. The purpose of DXGI is to expose capabilities that are common to all versions of Direct3D, decoupling them from the specific API version.

For example, these functionalities include:

- **Enumerating** available **device adapters**, **graphics modes**, and **display outputs**
- Querying for **hardware capabilities**
- Creating and destroying **swap chains** and **presenting** **rendered frames** to display outputs

To keep things simple, in the current example we avoid DXGI to enumerate all the 3D hardware and modes, and just use the default adapter. Instead DXGI will be used to create the Swap Chain.

Creating the DXGI Factory provides another example example of how COM objects are commonly used:

```cpp
Com<IDXGIFactory6*> dxgiFactory; 
CreateDXGIFactory2(DXGI_CREATE_FACTORY_DEBUG, IID_PPV_ARGS(&dxgiFactory));
```

### Different Interface Version in DirectX

Sometimes, you may notice a **number following  an interface name or function calls**, such as in the call to **CreateDXGIFactory2**. 

In DirectX, **different versions** of an interface are created to add new features. Each new version inherits from the previous one. For example, **IDXGIFactory2** inherits from **IDXGIFactory1**, which in turn inherits from **IDXGIFactory**. **IDXGIFactory2** includes all the methods from **IDXGIFactory1** and **IDXGIFactory**, plus any new methods.

When a new version of an interface is introduced, it typically includes a corresponding creation function, such as **CreateDXGIFactory2** for **IDXGIFactory2**.

As a general rule of thumb, the developer could use the most recent version of an interface unless there is a specific reason not to.

### Helper **CD3DX12_** Structs

Direct3D provides **helper structs** to facilitate the resources initialization. These structs are designed to simplify the process of populating the descriptors required for various Direct3D components, and are usually prefixed with **CD3DX12_**.

## A Simple D3D Application

Finally, we can start delving into code! 

Let’s outline the implementation:

- First, **setup the Graphic Pipeline**:
    - Create the Application Window
    - Initialize the D3D Device
    - Create the Command Queue, Allocator and List
    - Create the Swap Chain associated with the device using DXGI
    - Create and init the Fence for CPU/GPU synchronization
    - Define the Vertex Input Layout
    - Create the Vertex Buffer
    - Create the Root Signature
    - Compile the vertex and pixel shader
    - Configure the pipeline into a Pipeline State Object
- Then **run the Render Loop**, the window is closed:
    - Render the next frame
    - Wait for the frame to finish rendering

### Create the **Application Window**

To create a new window a description of its properties should be provided in the struct **WNDCLASSEX**. Then the class is registered and we can call the function ***CreateWindowEx*** to create a new instance of the window. We save the returned handle  because we’ll need it to create the swap chain.

```cpp
// Populate the struct that describes the properties of our application window
WNDCLASSEX windowClass = {};
windowClass.lpfnWndProc = wndMsgCallback; // Window callback procedure
// ... populate the struct with all the required properties

// Then we have to register the class before creating a window using it
if (!RegisterClassEx(&windowClass))
{
	// Handle error
}

// Finally, we can create the application window
hWnd = CreateWindowEx(
    exStyle,                      // Extended window style
    windowClass.lpszClassName,    // Class name
    L"D3D Example",               // Window name
    style,                        // Window style
    CW_USEDEFAULT,                // Horizontal position
    CW_USEDEFAULT,                // Vertical position
    wr.right - wr.left,           // Window width
    wr.bottom - wr.top,           // Window height
    NULL,                         // Parent window
    NULL,                         // Menu
    hInstance,                    // Instance handle
    NULL                          // Additional application data
);

if (!hWnd)
{
	// Handle error
}

```

Among the other parameters of **`WNDCLASSEX`** there is the name of the **callback procedure**, that a function whose purpose is **to handle all the system messages sent to the window**. These messages are fired whenever something involving the window happens, such as moving, resizing, or closing it. For now, the callback procedure is very simple: it posts the quit message when receives the **`WM_DESTROY`** message, sent when the window is closed. The quit message is then read from our render loop, as we’ll see later, which then proceeds to free the allocated resources and quit.

```cpp
/** Callback to handle messages directed to the application window */
LRESULT CALLBACK wndMsgCallback(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    switch (uMsg)
    {
        // Handle the window destruction message
        case WM_DESTROY:
            // Post a quit message and return
            PostQuitMessage(0);
            break;

        // Default handling for other messages
        default:
            return DefWindowProc(hWnd, uMsg, wParam, lParam);
    }
    return 0;
}
```

## Initialize the Graphic Pipeline

### The Adapter Initialization

First we have to initialize the adapter, calling ***D3D12CreateDevice*** passing **NULL** as the first argument, which instructs the function to use the **default device**. A **feature level** represents the set of all supported features from the API. In this case, we are requesting the latest feature level, 12.2.

```cpp
// HRESULT is used to determine the success or failure of a function call
HRESULT hResult;

// Create the Direct3D 12 device
// NULL specifies the default adapter
// D3D_FEATURE_LEVEL_12_1 requests the feature level 12.1
// IID_PPV_ARGS macro is used to retrieve the device interface
hResult = D3D12CreateDevice(NULL, D3D_FEATURE_LEVEL_12_2, IID_PPV_ARGS(&d3dDevice));

// Check if the device creation failed
if (FAILED(hResult))
    return; // Handle the error appropriately in a real application
```

### Create The Command Queue, The command allocator and the command list

To **record and submit commands** to the GPU, we need to create at least three objects: a command **allocator**, a command **list**, and a command **queue**. As we have seen in the previous lesson, the command allocator manages the memory for storing command lists, the command list is used to record commands that will be executed by the GPU. Finally, the command queue is responsible for submitting the command lists to the GPU for execution.

```cpp
// Populate the struct with the command queue required paramters
D3D12_COMMAND_QUEUE_DESC commandQueueDesc = {};  // Describe the command queue
commandQueueDesc.Type = D3D12_COMMAND_LIST_TYPE_DIRECT;  // Type of command list (Direct - can be used for all command types)
commandQueueDesc.Priority = D3D12_COMMAND_QUEUE_PRIORITY_NORMAL;  // Priority of the command queue
commandQueueDesc.Flags = D3D12_COMMAND_QUEUE_FLAG_NONE;  // No special flags
commandQueueDesc.NodeMask = 1;  // Single GPU node

// Create the command queue
hRes = d3dDevice->CreateCommandQueue(&commandQueueDesc, IID_PPV_ARGS(&commandQueue));  // Create the command queue
if (FAILED(hRes))
    return false;

// Create the command allocator, which will hold the data of the recorded commands from the list
hRes = d3dDevice->CreateCommandAllocator(D3D12_COMMAND_LIST_TYPE_DIRECT, IID_PPV_ARGS(&commandAllocator));  // Create the command allocator
if (FAILED(hRes))
    return false;

// Create the command list associated with the previous allocator
hRes = d3dDevice->CreateCommandList(0, D3D12_COMMAND_LIST_TYPE_DIRECT, commandAllocator.Get(), nullptr, IID_PPV_ARGS(&commandList));  // Create the command list
if (FAILED(hRes))
    return false;
```

In the following paragraphs, we will describe how to record commands in the list and draw a frame.

### Create the Swap Chain

Now it’s time to setup the swap chain. As we mentioned, the DXGI interface is used for this purpose.

As usual, we need to fill out a structure that describes the swap chain properties, in this case **`DXGI_SWAP_CHAIN_DESC1.`** Then we call the function **`*CreateSwapChainForHwnd*`** to creates an **`DXGISwapChain1`** . We have also to pass the **handle to the application window**, because that is the render target.

Here things get a bit tricky: we want to use the COM interface **`*IDXGISwapChain3` instead* `DXGISwapChain1` ,** because its **`*GetCurrentBackBufferIndex`*** method to track the current back buffer index is quite handy. The solutino is to convert the interface, using the ***As*** method, as the following snipped demonstrate:

```cpp
// Fill the description of the swap chain we are going to create. 
// We create a swap chain of type IDXGISwapChain1 so we can use the CreateSwapChainForHwnd method. 
// After that, we convert it to IDXGISwapChain3 for extended functionalities.

// Initialize the swap chain description
DXGI_SWAP_CHAIN_DESC1 swapChainDesc = {};
swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;   // Buffer usage for rendering
swapChainDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;             // Pixel format: 8 bits per channel including alpha
swapChainDesc.BufferCount = 2;                                 // Double buffering
swapChainDesc.Width = kApplicationWindowWidth;                 // Width of the back buffers
swapChainDesc.Height = kApplicationWindowHeight;               // Height of the back buffers
swapChainDesc.SampleDesc.Count = 1;                            // No anti-aliasing
swapChainDesc.SampleDesc.Quality = 0;                          // No anti-aliasing quality
swapChainDesc.AlphaMode = DXGI_ALPHA_MODE_UNSPECIFIED;         // Default behavior for blending pixels with alpha
swapChainDesc.Scaling = DXGI_SCALING_STRETCH;                  // Stretch the back buffer to fit the display area
swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_FLIP_DISCARD;      // Discard the back buffer content after presenting
swapChainDesc.Flags = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH;  // Allow switching between different display modes

// Create a swap chain of type IDXGISwapChain1
ComPtr<IDXGISwapChain1> swapChain1;
dxgiFactory->CreateSwapChainForHwnd(commandQueue.Get(), hWnd, &swapChainDesc, nullptr, nullptr, &swapChain1);

// Query for the IDXGISwapChain3 interface to use its extended methods
ComPtr<IDXGISwapChain3> swapChain3;
swapChain1.As(&swapChain3);
```

Two fields of **`DXGI_SWAP_CHAIN_DESC1`  need** a better explanation:

- **DXGI_FORMAT_R8G8B8A8_UNORM**: the format of the back buffer: 8 bits for each channel, including alpha. `UNORM` instead stands for Unsigned Normalized Integer, this values are floats in the range [0.0, 1.0].
- **DXGI_SWAP_EFFECT_FLIP_DISCARD**: this setting means that the content of the back buffer is discarded after it is presented. We don’t need to retain the previous frames, which can help improve performance.

The new swap chain automatically allocates the render target buffers. However, we’ll **later need to refer to these buffers**, for example, when flipping them. So we need to create the proper descriptors for them and the descriptor heap to store them.

```cpp
// Create a Descriptor Heap of type RTV to hold the two Render Target Views for the back buffers in the swap chain
D3D12_DESCRIPTOR_HEAP_DESC heapDesc = {};
heapDesc.NumDescriptors = 2;                            // Number of descriptors
heapDesc.Type = D3D12_DESCRIPTOR_HEAP_TYPE_RTV;         // Type of descriptor heap
heapDesc.Flags = D3D12_DESCRIPTOR_HEAP_FLAG_NONE;       // Flags (none in this case)

hRes = d3dDevice->CreateDescriptorHeap(&heapDesc, IID_PPV_ARGS(&rtvDescriptorHeap));
if (FAILED(hRes))
    return false; // Handle error appropriately

// The descriptor size could vary based on the specific GPU, so get this info from the device
descriptorRTVSize = d3dDevice->GetDescriptorHandleIncrementSize(D3D12_DESCRIPTOR_HEAP_TYPE_RTV);

// Create the two Render Target Views for the buffers in the swap chain
CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHeapHandle_CPU(rtvDescriptorHeap->GetCPUDescriptorHandleForHeapStart());
for (UINT i = 0; i < 2; i++)
{
    // Get the buffer from the swap chain and create a Render Target View for it
    hRes = swapChain->GetBuffer(i, IID_PPV_ARGS(&renderBuffers[i]));
    if (FAILED(hRes))
        return false; // Handle error appropriately

    // Create the RTV in the descriptor heap
    d3dDevice->CreateRenderTargetView(renderBuffers[i].Get(), nullptr, rtvHeapHandle_CPU);

    // Move to the next descriptor location in the heap
    rtvHeapHandle_CPU.Offset(1, descriptorRTVSize);
}
```

With this, the initialization of the swap chain and its descriptors is complete.

### Create and Init the Fence

Now, we create a the **fence** and initialize it with 0. Its value is used to synchronize the CPU with the GPU.

```cpp
// Create a fence, its value is used for CPU/GPU synchronization
hRes = d3dDevice->CreateFence(0, D3D12_FENCE_FLAG_NONE, IID_PPV_ARGS(&fence));
if (FAILED(hRes))
{
	// Handle error
}
```

### Defining the Vertex Input Layout

In this first example, our vertices only store the vertex position. Future lessons add more properties such as color, normals, and texture coordinates.

Let’s define our **vertex input layout** using the **`D3D12_INPUT_ELEMENT_DESC`** structure, which will later be bound to the pipeline. Each row of the element description corresponds to an input to the vertex shader, but in this case, it’s only the position. The semantic name  ‘POSITION’ is used later from the vertex shader to identify the inputs.

```cpp
// Define the vertex input layout, only the position in this simple example
D3D12_INPUT_ELEMENT_DESC vertexElementDesc[] =
{
    {
        "POSITION",                              // Semantic name: identifies the purpose of the data in the vertex buffer (POSITION in this case)
        0,                                       // Semantic index: used if there are multiple elements with the same semantic (e.g., POSITION0, POSITION1)
        DXGI_FORMAT_R32G32B32_FLOAT,              // Format: the data type of the element (32-bit float with 3 components for X, Y, Z)
        0,                                       // Input slot: specifies which input slot (vertex buffer) this element comes from (0 for this example)
        0,                                       // Aligned byte offset: offset in bytes from the start of the vertex structure (0 since POSITION is the first element)
        D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, // Input slot class: specifies if the data is per-vertex or per-instance
        0                                        // Instance data step rate: used for instanced rendering, 0 means no instancing
    }
};
```

On the CPU side each vertex will be stored in a struct that match the vertex layout.

```cpp
struct Vertex
{
	XMFLOAT3 position;
};
```

Here we are using the type **XMFLOAT3**, defined in **DirectXMath.h** that hold three floats, but we should have defined a new struct of three float as well.

### Create the Vertex Buffer

At this point, we need to define the vertices of the triangle and move them to a memory region that the GPU can read from. Note that the triangle vertices are in **clockwise winding** order, which is the order we are going to set in our pipeline.

To copy data from the CPU to **GPU accessible memory**, a resource of **type `D3D12_HEAP_TYPE_UPLOAD`** should be created. A little note here: using the **upload heap** directly is not efficient, and data that do not change should be moved to the **default heap**. For simplicity, we avoid doing that in this example, but we will address it in the next lessons.

We copy the data from the CPU memory to the GPU by mapping the vertex buffer to the CPU address space, then copying the triangle data into the GPU vertex buffer, and finally unmapping it.

Next, we need to create a **Vertex Buffer View** that describes how the data is arranged in the buffer and its purpose. This view doesn’t need to reside in a descriptor heap.

```cpp
// Define the vertices of the triangle in clockwise winding order
Vertex triangleVertices[] =
{
    { { 0.0f,  0.5f, 0.0f } }, // Vertex 1: Top
    { { 0.5f, -0.5f, 0.0f } }, // Vertex 2: Right
    { {-0.5f, -0.5f, 0.0f } }  // Vertex 3: Left
};

// Describe the vertex buffer
CD3DX12_HEAP_PROPERTIES heapProps(D3D12_HEAP_TYPE_UPLOAD);
CD3DX12_RESOURCE_DESC bufferDesc = CD3DX12_RESOURCE_DESC::Buffer(sizeof(triangleVertices));

// Create the vertex buffer resource in the upload heap
ComPtr<ID3D12Resource> vertexBuffer;
HRESULT hRes = d3dDevice->CreateCommittedResource(
    &heapProps,
    D3D12_HEAP_FLAG_NONE,
    &bufferDesc,
    D3D12_RESOURCE_STATE_GENERIC_READ,
    nullptr,
    IID_PPV_ARGS(&vertexBuffer)
);
if (FAILED(hRes))
    return false; // Handle error appropriately

// Map the vertex buffer to the CPU address space
UINT8* vertexDataBegin;
CD3DX12_RANGE readRange(0, 0); // We do not intend to read from this resource on the CPU
hRes = vertexBuffer->Map(0, &readRange, reinterpret_cast<void**>(&vertexDataBegin));
if (FAILED(hRes))
{
	// handle error
}

// Copy the triangle data to the vertex buffer
memcpy(vertexDataBegin, triangleVertices, sizeof(triangleVertices));

// Unmap the vertex buffer, making the data available to the GPU
vertexBuffer->Unmap(0, nullptr);

// Create the vertex buffer view
D3D12_VERTEX_BUFFER_VIEW vertexBufferView = {};
vertexBufferView.BufferLocation = vertexBuffer->GetGPUVirtualAddress();
vertexBufferView.StrideInBytes = sizeof(Vertex);
vertexBufferView.SizeInBytes = sizeof(triangleVertices);
```

### Creating The Root Signature

The root signature defines **which resources are bound to the rendering pipeline**. However, for now, we have no resources except for the vertex buffer, which is bound adding a specific command in the command list. Therefore, we only need an empty root signature:

```cpp
// Create an empty root signature
CD3DX12_ROOT_SIGNATURE_DESC rootSignatureDesc;
// Initialize the root signature descriptor with no parameters
rootSignatureDesc.Init(0, nullptr, 0, nullptr, D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT);

ComPtr<ID3DBlob> signature; // Blob to hold the serialized root signature
ComPtr<ID3DBlob> error; // Blob to hold error messages, if any

// Serialize the root signature description into a blob
D3D12SerializeRootSignature(&rootSignatureDesc, D3D_ROOT_SIGNATURE_VERSION_1, &signature, &error);
if (error)
{
    // Handle error (error blob contains the error message)
}

// Create the root signature from the serialized blob
HRESULT hRes = d3dDevice->CreateRootSignature(0, signature->GetBufferPointer(), signature->GetBufferSize(), IID_PPV_ARGS(&rootSignature));
if (FAILED(hRes))
{
    // Handle error
}
```

**ID3DBlob** is a binary large object used to store data like compiled shader bytecode, root signatures, and error messages. It is commonly used in Direct3D to handle and transfer large blocks of binary data.

## The Vertex and Pixel shader

Shaders are the fully programmable parts of this pipeline. The vertex shader is executed just after the **input assembler** stage, as explained in the previous lesson. The inputs are organized and bound to the shader using the **semantics** defined in the **input vertex layout**. The shaders are written in  **HLSL (High-Level Shader Language)** and stored in the file *shaders.hlsl*.

Here we implement two very simple shaders:

- **Vertex Shader**: takes the 2D position as input and just outputs it.
- **Pixel Shader**: assign the white color to each fragment.

Note that **`position`** has the semantic **`POSITION`**, the same used in the vertex input layout. The vertex shader outputs these positions using the **`SV_POSITION`** semantic, where **`SV`** stands for **System Value**, indicating that these are the transformed vertex positions to be used by subsequent pipeline stages. 

```cpp
// Vertex Shader
void VSMain(float4 position : POSITION, out float4 outPosition : SV_POSITION)
{
    // Pass the position directly to the output
    outPosition = position;
}
```

Then the following pixel shader is executed for each pixel of the triangle. It just returns white as color. 
 In the pixel shader, the **`SV_Target`** semantic specifies that the output color is the final color for the pixel. 

```cpp
// Pixel Shader
float4 PSMain() : SV_Target
{
    // Just output white
    return float4(1.f, 1.f, 1.f, 1.f);
}
```

### Compiling the shaders

Now we compile the vertex and pixel shaders from the file`shaders.hlsl`. The function **`D3DCompileFromFile`** compiles them.**`VSMain`** and **`PSMain`** are the shader entry points. The compiled shader bytecode is stored in `ID3DBlob` objects. These will be bound later to the pipeline.

```cpp
// Compile the vertex shader from the HLSL file
ComPtr<ID3DBlob> vertexShader;  // Blob to hold the compiled vertex shader bytecode
HRESULT hRes = D3DCompileFromFile(
    L"shaders.hlsl",         // Path to the HLSL file
    nullptr,                 // Optional array of D3D_SHADER_MACRO structures
    nullptr,                 // Optional include interface for handling #include directives
    "VSMain",                // Entry point for the vertex shader
    "vs_5_0",                // Target profile for the vertex shader
    0,                       // Compile options (set to 0 for default options)
    0,                       // Effect compile options (set to 0 for default options)
    &vertexShader,           // Blob to hold the compiled shader bytecode
    nullptr                  // Blob to hold error messages, if any
);
if (FAILED(hRes))
{
    // Handle error appropriately
}
```

### Create the Pipeline State Object

Finally, we set up the **Pipeline State Object (PSO)**, which defines the configuration of the graphics pipeline. Here we put **everything together**: the input layout, root signature, shaders, the rasterizer stage configuration, the blend configuuration, and more. In this simple example, we disable depth and stencil testing.

```cpp
// Set up Pipeline State Object (PSO)
D3D12_GRAPHICS_PIPELINE_STATE_DESC psoDesc = {};                                              // Initialize the PSO description structure
psoDesc.InputLayout = { vertexElementDesc, _countof(vertexElementDesc) };                      // Specify the input layout
psoDesc.pRootSignature = rootSignature.Get();                                                  // Set the root signature
psoDesc.VS = { reinterpret_cast<UINT8*>(vertexShader->GetBufferPointer()), vertexShader->GetBufferSize() };  // Attach the vertex shader
psoDesc.PS = { reinterpret_cast<UINT8*>(pixelShader->GetBufferPointer()), pixelShader->GetBufferSize() };    // Attach the pixel shader
psoDesc.RasterizerState = CD3DX12_RASTERIZER_DESC(D3D12_DEFAULT);                              // Configure the rasterizer with default values
psoDesc.RasterizerState.FrontCounterClockwise = FALSE;                                         // Set clockwise winding order on the rasterizer
psoDesc.BlendState = CD3DX12_BLEND_DESC(D3D12_DEFAULT);                                        // Set the blend state to default
psoDesc.DepthStencilState.DepthEnable = FALSE;                                                 // Disable depth testing
psoDesc.DepthStencilState.StencilEnable = FALSE;                                               // Disable stencil testing
psoDesc.SampleMask = UINT_MAX;                                                                 // Set the sample mask to allow all samples
psoDesc.PrimitiveTopologyType = D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE;                        // Define the primitive topology as triangles
psoDesc.NumRenderTargets = 1;                                                                  // Specify the number of render targets
psoDesc.RTVFormats[0] = DXGI_FORMAT_R8G8B8A8_UNORM;                                            // Set the format of the render target to match the swap chain buffer
psoDesc.SampleDesc.Count = 1;                                                                  // Set the sample description to use one sample per pixel
HRESULT hRes = d3dDevice->CreateGraphicsPipelineState(&psoDesc, IID_PPV_ARGS(&pipelineState)); // Create the pipeline state object
if (FAILED(hRes))
    return false;
```

## The Render Loop

The render loop is very simple: 

- Run until the Quit message is received
- Use **`PeekMessage`** to handle messages and **`DispatchMessage`** to forward them to the window callback
- Call **`renderFrame`** to draw the current frame
- Call  **`waitForFrameCompletion`** to ensure the frame is fully processed before moving to the next one

When the ***quit*** message is received, quit the loop

```cpp
MSG msg = { WM_NULL };
while (msg.message != WM_QUIT)
{
    // Process window messages without blocking the loop
    if (PeekMessage(&msg, 0, 0, 0, PM_REMOVE))
    {
        TranslateMessage(&msg);
        DispatchMessage(&msg);  // Dispatch the message to the window callback procedure
    }

    renderFrame();  // Render the current frame
    waitForFrameCompletion();  // Ensure the frame is fully processed
}

waitForFrameCompletion();  // Final wait for frame processing completion
```

### Rendering the frame

The **`renderFrame`** function is where the frame is rendered. It’s quite long, so we are going to outline it:

- First, we **set the viewport**, which defines the area of the render target to draw into. In this case, it covers the entire back buffer.
- Next, we **set the scissor rectangle**, which is the portion of the viewport where rendering is allowed. Again, it covers the entire back buffer.
- Then, we **transition the back buffer from its present state to a render target state and clear it** with a white color to prepare for drawing.
- **Set the vertex buffer**
- **Draw a triangle**, ***DrawInstanced*** here uses the first three elements of the vertex buffer to draw a single primitive, i.e. a triangle
- After drawing, the **back buffer is transitioned** from the render target state **to the present state**
- Close and **submit the command list**
- **Present the frame**

```cpp
// [comment]
// This function renders a new frame by filling and executing the command list, and then presenting the back buffer.
// [/comment]
void renderFrame()
{
    // Get the current back buffer index
    currentBackBufferIndex = swapChain->GetCurrentBackBufferIndex();

    // Reset the command allocator; it can only be reset once all commands in the associated command list have been executed
    commandAllocator->Reset();

    // Reset the command list; it can be reset as soon as it has been submitted using ExecuteCommandList()
    commandList->Reset(commandAllocator.Get(), pipelineState.Get());

    // Set the root signature for the command list
    commandList->SetGraphicsRootSignature(rootSignature.Get());

    // Set up the viewport to cover the entire back buffer
    commandList->RSSetViewports(1, &kViewPort);

    // Set up the scissor rectangle to cover the entire back buffer
    commandList->RSSetScissorRects(1, &kScissorRect);

    // Transition the current back buffer from present state to render target state
    D3D12_RESOURCE_BARRIER barrier = CD3DX12_RESOURCE_BARRIER::Transition(renderBuffers[currentBackBufferIndex].Get(), D3D12_RESOURCE_STATE_PRESENT, D3D12_RESOURCE_STATE_RENDER_TARGET);
    commandList->ResourceBarrier(1, &barrier);

    // Set the back buffer as the render target
    CD3DX12_CPU_DESCRIPTOR_HANDLE rtvHandle(rtvDescriptorHeap->GetCPUDescriptorHandleForHeapStart(), currentBackBufferIndex, descriptorRTVSize);
    commandList->OMSetRenderTargets(1, &rtvHandle, FALSE, nullptr);

    // Clear the render target with a default background color (white)
    commandList->ClearRenderTargetView(rtvHandle, kDefaultBackgroundColor, 0, nullptr);

    // Set the primitive topology to triangles
    commandList->IASetPrimitiveTopology(D3D_PRIMITIVE_TOPOLOGY_TRIANGLELIST);

    // Set the vertex buffer
    commandList->IASetVertexBuffers(0, 1, &vertexBufferView);

    // Draw the triangle
    commandList->DrawInstanced(3, 1, 0, 0);

    // Transition the back buffer from render target state to present state
    barrier = CD3DX12_RESOURCE_BARRIER::Transition(renderBuffers[currentBackBufferIndex].Get(), D3D12_RESOURCE_STATE_RENDER_TARGET, D3D12_RESOURCE_STATE_PRESENT);
    commandList->ResourceBarrier(1, &barrier);

    // Close the command list after all commands have been added
    commandList->Close();

    // Execute the command list; ExecuteCommandLists requires a pointer to an array of command lists
    ID3D12CommandList* ppCommandLists[] = { commandList.Get() };
    commandQueue->ExecuteCommandLists(1, ppCommandLists);

    // Present the frame
    swapChain->Present(1, 0);
}
```

### Wait for the previous frame to finish

We’ll use the **`waitForFrameCompletion`** function to **synchronize** the CPU and GPU by using a **fence**. Is used a very simple synchronization method that **blocks the CPU until the GPU finishes** executing its commands. In more advanced strategies, the CPU continues working on the next frame without waiting for the GPU to finish the previous one. However, for the purpose of this simple application, this approach is enough. 

The function first retrieves the current fence value, then **increments** it and **signals** the command queue to update the fence to the new value. An event handle is created and set to be fired once the GPU reaches the updated fence value. At this point, the function waits for this event to be triggered, indicating that the GPU has finished processing the current frame, and we can start the next iteration.

```cpp
void waitForFrameCompletion()
{
    // Get the current fence value, which represents the GPU's completed work up to this point
    UINT64 fenceValue = fence->GetCompletedValue();

    // Enqueue a signal command to update the fence value to the next value
    commandQueue->Signal(fence.Get(), ++fenceValue);

    // Create an event handle that will be used to wait for the GPU to signal the fence
    HANDLE fenceEvent = CreateEvent(nullptr, FALSE, FALSE, nullptr);
    if (fenceEvent == nullptr)
        return; // Handle the error appropriately in a real application

    // Set the event to be signaled when the GPU has completed the work up to the new fence value
    fence->SetEventOnCompletion(fenceValue, fenceEvent);

    // Wait for the event to be signaled, indicating the GPU has completed the work
    WaitForSingleObject(fenceEvent, INFINITE);

    // Close the event handle to free resources
    CloseHandle(fenceEvent);
}
```

## Conclusion

So we ended our first app in Direct3D, and it has certainly been daunting! From now on, things will get smoother. In the next lessons, we’ll concentrate on more specific aspects of rendering, revisiting the things we have already used in more detail and adding new features to our application.
	
*That's it for now. See you next time !!!*
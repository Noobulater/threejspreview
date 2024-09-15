LIVE WEBSITE HERE

BLENDER SYNCHRONIZATION HERE

# Three.js WebSocket and File Loader Integration Guide

This guide explains how to lift the WebSocket connection and file loading functionality (GLB/GLTF and HDR) from the provided code and integrate it into a different Three.js-based application.

## Prerequisites
Before you begin, ensure that your application meets the following requirements:
- Your application uses Three.js.
- You have a Three.js scene with a camera, renderer, and basic scene setup (or are ready to set this up).
- You have a working environment capable of handling JavaScript modules (ES6 imports).

## Features to Integrate

This integration focuses on the following features from the provided code:
1. **WebSocket Connection**: Enables real-time communication to receive GLB/GLTF models over WebSocket.
2. **File Loading (GLB/GLTF & HDR)**: Allows for drag-and-drop or WebSocket-based loading of GLB/GLTF files (for 3D models) and HDR files (for environment maps).

### Steps for Integration

### 1. Set Up WebSocket Connection

#### Lift the WebSocket Setup:
The provided code establishes a WebSocket connection to receive GLB files. Below is the WebSocket setup that you need to integrate into your own application.

```
let ws;
const reconnectInterval = 5000; // 5 seconds
let isConnected = false;

function connect() {
    ws = new WebSocket('ws://localhost:42069'); // Replace with your server URL

    ws.onopen = function () {
        isConnected = true;
        console.log("Connected to WebSocket server!");
    };

    ws.onmessage = function (event) {
        console.log('Message from server:', event.data);
        if (typeof event.data === 'object') {
            const blob = new Blob([event.data], { type: 'model/gltf-binary' });
            const file = new File([blob], "model.glb", { type: 'model/gltf-binary' });
            loadFile(file);  // Call the file loader
            console.log('Received GLB file blob');
        }
    };

    ws.onclose = function () {
        isConnected = false;
        console.log("Disconnected from WebSocket server.");
    };

    ws.onerror = function (error) {
        console.log("Error connecting to WebSocket server.");
        console.error('WebSocket error:', error);
        ws.close();
        ws = undefined;
    };
}

function checkConnection() {
    if (!isConnected) {
        console.log("Attempting to reconnect...");
        connect(); // Attempt to reconnect if not connected
    }
}

// Start WebSocket connection
connect();
setInterval(checkConnection, reconnectInterval);
```

#### How to Integrate:
- Ensure that the URL (`ws://localhost:42069`) is replaced with your actual WebSocket server's URL.
- Call the `connect()` function when your Three.js application starts, so the WebSocket connection is immediately established.

### 2. Implement File Loading

The file loading functionality allows you to load `.glb`, `.gltf`, and `.hdr` files into the scene. Here's how to implement the file loading:

#### Lift the File Loading Logic:
Add this file loading logic to handle the drag-and-drop or WebSocket-based loading of GLTF/GLB models and HDR environment maps:

```
function loadFile(file) {
    const filename = file.name;
    const extension = filename.split('.').pop().toLowerCase();
    const reader = new FileReader();

    reader.addEventListener('progress', function(event) {
        const progress = Math.floor((event.loaded / event.total) * 100) + '%';
        console.log(`Loading ${filename}: ${progress}`);
        // You can add a progress bar update here if needed
    });

    switch (extension) {
        case 'hdr':
            const url = URL.createObjectURL(file);
            const rgbeLoader = new RGBELoader();
            rgbeLoader.setCrossOrigin("anonymous");
            rgbeLoader.load(url, texture => {
                texture.mapping = THREE.EquirectangularReflectionMapping;
                scene.background = scene.environment = texture;
                URL.revokeObjectURL(url);
            }, undefined, console.error);
            break;

        case 'glb':
        case 'gltf':
            reader.addEventListener('load', function (event) {
                const contents = event.target.result;
                const loader = new GLTFLoader();
                loader.parse(contents, '', gltf => {
                    loadGLTF(gltf, anchor); // Assuming you have a function to handle GLTF loading
                });
            }, false);
            reader.readAsArrayBuffer(file);
            break;
    }
}
```

#### Drag-and-Drop Event Setup:
Set up drag-and-drop functionality for loading files by adding event listeners:

```
document.addEventListener('dragover', function(event) {
    event.preventDefault();  // Prevent default behavior (open file)
});

document.addEventListener('drop', function(event) {
    event.preventDefault();  // Prevent default behavior (open file)
    const file = event.dataTransfer.files[0];  // Get the first file
    if (file) {
        loadFile(file);
    }
});
```

#### How to Integrate:
- Ensure that `loadGLTF(gltf, anchor)` correctly integrates with your existing Three.js scene to add the loaded 3D model to the scene.
- Use the `RGBELoader` for environment maps, adding HDR files as backgrounds and environment maps.

### 3. Progress Bar (Optional)

If you want to integrate the progress bar for file loading, make sure to update the progress display within the `loadFile` function:

```
reader.addEventListener('progress', function(event) {
    const progress = Math.floor((event.loaded / event.total) * 100) + '%';
    document.getElementById('progress').style.width = progress;
});
```

### 4. Resize Handling

Ensure that your application can handle window resizing by adding this code:

```
window.addEventListener('resize', onWindowResize, false);

function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
}
```

### Final Integration Checklist

1. **WebSocket Setup**:
   - Integrate the `connect()`, `checkConnection()`, and WebSocket handlers in your application to receive GLB/GLTF models via WebSocket.
   
2. **File Loading**:
   - Ensure that `loadFile(file)` is called when a file is dropped onto the page or received via WebSocket.
   - Handle GLTF/GLB models and HDR environment maps in your existing Three.js scene.

3. **Optional**: Add progress bar functionality by updating the DOM during file loading.

4. **Resize Handling**: Ensure that the camera and renderer respond to window resizing to maintain correct aspect ratios.

---

By following these steps, you can successfully lift the WebSocket and file loading features from the provided code and integrate them into your Three.js application.

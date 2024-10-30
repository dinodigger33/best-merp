<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3D Game with Three.js</title>
    <style>
        body { margin: 0; overflow: hidden; }
    </style>
</head>
<body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // Set up scene, camera, and renderer
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        // Create a ground plane
        const planeGeometry = new THREE.PlaneGeometry(50, 50);
        const planeMaterial = new THREE.MeshStandardMaterial({ color: 0xaaaaaa });
        const plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -Math.PI / 2; // Rotate plane to be horizontal
        plane.receiveShadow = true;
        scene.add(plane);

        // Create a player cube
        const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
        const cubeMaterial = new THREE.MeshStandardMaterial({ color: 0xff0000 });
        const playerCube = new THREE.Mesh(cubeGeometry, cubeMaterial);
        playerCube.castShadow = true;
        playerCube.position.y = 0.5; // Position above the ground
        scene.add(playerCube);

        // Lighting
        const ambientLight = new THREE.AmbientLight(0x404040);
        scene.add(ambientLight);

        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
        directionalLight.position.set(5, 10, 7.5);
        directionalLight.castShadow = true;
        scene.add(directionalLight);

        // Camera position
        camera.position.z = 5;
        camera.position.y = 5;
        camera.lookAt(0, 0, 0);

        // Variables for controls
        const playerSpeed = 0.1;
        const keys = {};

        // Handle keyboard events for movement
        document.addEventListener("keydown", (event) => {
            keys[event.key] = true;
        });

        document.addEventListener("keyup", (event) => {
            keys[event.key] = false;
        });

        // Game loop
        function animate() {
            requestAnimationFrame(animate);

            // Player movement
            if (keys["ArrowUp"] || keys["w"]) playerCube.position.z -= playerSpeed;
            if (keys["ArrowDown"] || keys["s"]) playerCube.position.z += playerSpeed;
            if (keys["ArrowLeft"] || keys["a"]) playerCube.position.x -= playerSpeed;
            if (keys["ArrowRight"] || keys["d"]) playerCube.position.x += playerSpeed;

            // Rotate cube for visual effect
            playerCube.rotation.x += 0.01;
            playerCube.rotation.y += 0.01;

            renderer.render(scene, camera);
        }

        animate();

        // Handle window resizing
        window.addEventListener("resize", () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
        });
    </script>
</body>
</html>

   
   


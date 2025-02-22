---
permalink: /bdo-fish-position/
title: " "
exceprt: ""
layout: single

header:
  overlay_image: /assets/images/bdo_fish_zones/Other Regions - Coast Surfperch Default_Preview.webp
author_profile: false

sidebar:
  - title: "How To Use"
    text: "1. Make a waypoint exactly at your position, or where your float lands.\n2. Save the waypoint into your favorites through the worldmap and give it a name you remember for step 5.\n3. Got to \n Documents/Black Desert/UserCache/ACCOUNTID folder.\n4. Open the file named \"gamevariable.xml\" in any text editor of your choice.\n5. In that file search for the name you used in step 2.\n6. Copy the X and the Z(!) coordinates from the bookmark into the boxes to the right and press the button."


---
# Black Desert Fish Region Checker
<html>

<head>
    <style>
        .input-container {
            margin: 20px 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .input-field {
            margin: 10px;
            padding: 5px;

        }

        #result {
            margin-top: 20px;
            padding: 10px;
            border: 1px solid #ccc;
            display: none;
            margin-bottom: 20px;
        }

        .result-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 100%;
            max-width: 800px;
            margin: 0 auto;
        }

        .color-info {
            width: 100%;
        }

        .image-container {
            width: 100%;
            text-align: center;
        }

        #colorImage {
            max-width: 100%;
            height: auto;
            min-width: 500px;
            display: none;
            border: 1px solid #ccc;
            margin-top: 20px;
        }
    </style>
</head>

<body>
    <div class="input-container">
        <input type="number" step="0.01" id="input1" class="input-field" placeholder="X Coordinate">
        <input type="number" step="0.01" id="input2" class="input-field" placeholder="Z Coordinate">
        <button onclick="processCoordinates()">Check Fishing Region</button>
    </div>

    <div class="result-container">
        <div id="result" class="color-info"></div>
        <div class="image-container">
            <img id="colorImage" alt="Color representation">
        </div>
    </div>

    <script>

        const img = new Image();
        img.crossOrigin = "Anonymous"; // Enable CORS if image is from different domain
        img.src = "/assets/images/bdo_fish_zones/fishingmap.png";


        let colorData = [];

        fetch('/assets/tables/bdo_fish_zones/fishingtable.csv')
            .then(response => response.text())
            .then(data => {
                // Parse CSV
                const rows = data.split('\n');
                rows.forEach(row => {
                    if (row.trim()) {
                        const [r, g, b, name] = row.split(';');
                        colorData.push({
                            r: parseInt(r),
                            g: parseInt(g),
                            b: parseInt(b),
                            name: name
                        });
                    }
                });
            })
            .catch(error => console.error('Error loading color data:', error));

        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');

        img.onload = function () {
            canvas.width = img.width;
            canvas.height = img.height;
            ctx.drawImage(img, 0, 0);
        };

        const LEFT = -160;
        const TOP = 160;

        function worldPositionToPixel(worldX, worldZ) {

            let sectorX = worldX / 12800;
            let sectorZ = worldZ / 12800;

            sectorX -= LEFT;
            sectorZ -= TOP;

            let pixelX = sectorX / 0.0235294122248888;
            let pixelY = sectorZ / 0.0235294122248888;

            pixelY = -(pixelY + 1);

            pixelX = Math.max(0, Math.min(11560 - 1, pixelX));
            pixelY = Math.max(0, Math.min(10540 - 1, pixelY));
            
            return {
                x: Math.round(pixelX),
                y: Math.round(pixelY)
            };
        }
        function findClosestColor(r, g, b) {
            let closestColor = null;
            let minDistance = Infinity;

            colorData.forEach(color => {
                const distance = Math.sqrt(
                    Math.pow(r - color.r, 2) +
                    Math.pow(g - color.g, 2) +
                    Math.pow(b - color.b, 2)
                );

                if (distance < minDistance) {
                    minDistance = distance;
                    closestColor = color;
                }
            });

            return closestColor;
        }
        function loadColorImage(colorName) {
            const colorImage = document.getElementById('colorImage');

            const imagePath = `/assets/images/bdo_fish_zones/${colorName}_Preview.webp`;

            colorImage.src = imagePath;
            colorImage.style.display = 'block';


            colorImage.onerror = function () {
                colorImage.style.display = 'none';
                console.error(`Image not found for color: ${colorName}`);
            };
        }
        function processCoordinates() {
            const num1 = parseFloat(document.getElementById('input1').value);
            const num2 = parseFloat(document.getElementById('input2').value);

            if (isNaN(num1) || isNaN(num2)) {
                alert('Please enter valid numbers');
                return;
            }

            const coords = worldPositionToPixel(num1, num2);

            try {

                const pixelData = ctx.getImageData(coords.x, coords.y, 1, 1).data;
                const r = pixelData[0];
                const g = pixelData[1];
                const b = pixelData[2];
                const color = `rgb(${r}, ${g}, ${b})`;

                const closestColor = findClosestColor(r, g, b);
                const colorName = closestColor ? closestColor.name : 'Unknown';

                const resultDiv = document.getElementById('result');
                resultDiv.style.display = 'block';
                //resultDiv.style.backgroundColor = color;
                resultDiv.innerHTML = `Your current fishing region: ${colorName}`;

                if (colorName !== 'Unknown') {
                    loadColorImage(colorName);
                }
            } catch (e) {
                alert('Error: Coordinates may be outside image boundaries');
            }
        }
    </script>
</body>

</html>
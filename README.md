# pr
react code and u can jus setup tailwind css in the local system  

import React, { useRef, useState } from "react";

export default function ImageProcessor() {
  const [originalSrc, setOriginalSrc] = useState(null);
  const [processedSrc, setProcessedSrc] = useState(null);
  const [pixelInfo, setPixelInfo] = useState("");
  const [neighborhoodSize, setNeighborhoodSize] = useState(3);
  const canvasRef = useRef(null);

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    const url = URL.createObjectURL(file);
    setOriginalSrc(url);
    setProcessedSrc(null);

    const img = new Image();
    img.onload = () => processImage(img);
    img.src = url;
  };

  const processImage = (img) => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
    canvas.width = img.width;
    canvas.height = img.height;

    ctx.drawImage(img, 0, 0);
    let imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    let data = imageData.data;
    const width = canvas.width;
    const height = canvas.height;
    const size = neighborhoodSize;
    const radius = Math.floor(size / 2);

    // Convert to grayscale first
    for (let i = 0; i < data.length; i += 4) {
      const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
      data[i] = data[i + 1] = data[i + 2] = avg;
    }

    // Clone for reading
    const copy = new Uint8ClampedArray(data);

    // Apply average blur (neighborhood filter)
    for (let y = radius; y < height - radius; y++) {
      for (let x = radius; x < width - radius; x++) {
        let sum = 0;
        let count = 0;

        for (let dy = -radius; dy <= radius; dy++) {
          for (let dx = -radius; dx <= radius; dx++) {
            const px = (y + dy) * width + (x + dx);
            sum += copy[px * 4]; // grayscale so any channel is fine
            count++;
          }
        }

        const avg = sum / count;
        const idx = (y * width + x) * 4;
        data[idx] = data[idx + 1] = data[idx + 2] = avg;
      }
    }

    ctx.putImageData(imageData, 0, 0);
    setProcessedSrc(canvas.toDataURL());
  };

  const handleMouseMove = (e) => {
    const canvas = canvasRef.current;
    const rect = canvas.getBoundingClientRect();
    const x = Math.floor(e.clientX - rect.left);
    const y = Math.floor(e.clientY - rect.top);
    const ctx = canvas.getContext("2d");
    const data = ctx.getImageData(x, y, 1, 1).data;
    setPixelInfo(`X: ${x}, Y: ${y}, Gray: ${data[0]}`);
  };

  return (
    <div className="p-6 space-y-6 max-w-4xl mx-auto">
      <input
        type="file"
        accept="image/*"
        onChange={handleImageUpload}
        className="mb-4"
      />

      <div className="flex items-center gap-4">
        <label className="text-sm font-semibold">Neighborhood size:</label>
        <input
          type="number"
          value={neighborhoodSize}
          min={1}
          step={2}
          onChange={(e) => setNeighborhoodSize(parseInt(e.target.value))}
          className="border px-2 py-1 w-16"
        />
        <span className="text-xs text-gray-500">(odd only, e.g. 3, 5, 7)</span>
      </div>

      <div className="grid grid-cols-2 gap-6">
        {originalSrc && (
          <div>
            <p className="text-center mb-2 font-bold">Original</p>
            <img src={originalSrc} alt="Original" className="w-full" />
          </div>
        )}
        {processedSrc && (
          <div>
            <p className="text-center mb-2 font-bold">Blurred (Avg)</p>
            <canvas
              ref={canvasRef}
              onMouseMove={handleMouseMove}
              className="w-full border"
            />
            <p className="text-sm mt-2 text-center">{pixelInfo}</p>
          </div>
        )}
      </div>
    </div>
  );
}

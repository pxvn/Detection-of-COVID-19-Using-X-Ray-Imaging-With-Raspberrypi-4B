```python

import tflite_runtime.interpreter as tflite
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
def classify_image(image_path, model_path="model_unquant.tflite", labels_path="labels.txt"):
   
    
    """Classifies a grayscale image for COVID-19 using a TFLite model.

    Args:
        image_path: Path to the grayscale image file.
        model_path: Path to the TFLite model file (default: "model_unquant.tflite").
        labels_path: Path to the labels text file (default: "labels.txt").

    Returns:
        A dictionary containing:
            - original_image: The original RGB image (PIL.Image object).
            - inverted_image: The inverted RGB image (PIL.Image object).
            - predicted_label: The predicted label (string).
            - confidence: The confidence of the prediction (float).
    """

    # Load model and labels
    interpreter = tflite.Interpreter(model_path=model_path)
    interpreter.allocate_tensors()
    input_details = interpreter.get_input_details()[0]
    with open(labels_path, "r") as f:
        labels = [line.strip() for line in f.readlines()]

    # Preprocess image
    img_original = Image.open(image_path)
    img_rgb = Image.fromarray(np.stack((np.array(img_original),) * 3, axis=-1), "RGB")
    img = img_rgb.resize((input_details['shape'][1], input_details['shape'][2]))
    img_array = np.array(img) / 255.0
    img_array = np.expand_dims(img_array, axis=0).astype(input_details['dtype'])

    # Run inference
    interpreter.set_tensor(input_details['index'], img_array)
    interpreter.invoke()
    output_details = interpreter.get_output_details()[0]
    output_data = interpreter.get_tensor(output_details['index'])

    # Get prediction and confidence
    top_index = np.argmax(output_data)
    predicted_label = labels[top_index]
    confidence = output_data[0][top_index]

    # Create inverted image
    img_inverted = Image.fromarray(255 - np.array(img_rgb)).convert("RGB")

    return {
        "original_image": img_original,
        "inverted_image": img_inverted,
        "predicted_label": predicted_label,
        "confidence": confidence
    }

# Example Usage
image_path = "NC3.jpeg"
results = classify_image(image_path)

# Display images and results
fig, axs = plt.subplots(1, 2)
axs[0].imshow(results["original_image"])
axs[0].set_title("XRAY Image")
axs[1].imshow(results["inverted_image"])
axs[1].set_title("Inverted Image")

print(f"Predicted Label: {results['predicted_label']}")
print(f"Confidence: {results['confidence']:.2f}")

plt.show()```
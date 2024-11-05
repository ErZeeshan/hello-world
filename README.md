
# Brand Profile Image Generator Documentation

## Project Overview

This project is a web application that allows users to upload an image, select a background from predefined brand options, and generate a profile image with a brand-specific background and color ring. The application utilizes OpenAI's DALL-E inpainting feature to remove the image background and apply branding elements, all while sticking to OpenAI's ecosystem.

### Project Structure

- **Frontend**: Provides a user interface for image upload, background selection, and displaying the final branded image.
- **Backend**: A Flask application that handles image processing using OpenAI's API.

---

## Prerequisites

1. **Python** (with Flask installed)
2. **OpenAI API key** with access to DALL-E inpainting capabilities
3. **Basic HTML, JavaScript, and CSS** for the frontend

---

## Setup Instructions

### Step 1: Install Required Packages

Install Flask and other dependencies by running:

```bash
pip install flask requests openai
```

### Step 2: Obtain OpenAI API Key

Sign up or log in to [OpenAI](https://platform.openai.com) and get an API key. Replace `"YOUR_OPENAI_API_KEY"` in the backend code with your actual API key.

---

## Backend Code (Flask)

Create a file named `app.py` and add the following code.

```python
import os
import openai
from flask import Flask, request, jsonify, send_file
from PIL import Image
import requests
from io import BytesIO

app = Flask(__name__)

# Set up OpenAI API key
openai.api_key = "YOUR_OPENAI_API_KEY"

# Endpoint to handle image upload and processing
@app.route('/process_image', methods=['POST'])
def process_image():
    # Get the uploaded file and brand information
    file = request.files['image']
    brand_background = request.form.get("brand_background", "default background")
    brand_color = request.form.get("brand_color", "blue")
    
    # Save the uploaded image temporarily
    file_path = "uploaded_image.png"
    file.save(file_path)

    # Step 1: Remove the background using DALL-E inpainting
    try:
        # Load the image for inpainting
        with open(file_path, "rb") as img_file:
            response = openai.Image.create_edit(
                image=img_file,
                prompt="Remove the background around the central subject, leaving only the subject isolated with transparency.",
                mask="transparent",
                n=1,
                size="1024x1024",
            )
        
        # Download the edited image with transparent background
        edited_image_url = response['data'][0]['url']
        edited_image_response = requests.get(edited_image_url)
        img_with_transparency = Image.open(BytesIO(edited_image_response.content))

    except Exception as e:
        return jsonify({"error": f"Error in background removal: {str(e)}"}), 500

    # Step 2: Apply brand background and color ring
    try:
        # Convert the edited image to apply branding
        brand_prompt = f"Add a {brand_background} background behind the person and place a circular ring around the profile using {brand_color}."
        
        response = openai.Image.create_edit(
            image=BytesIO(edited_image_response.content),
            prompt=brand_prompt,
            n=1,
            size="1024x1024",
        )
        
        # Get the final branded image URL
        final_image_url = response['data'][0]['url']

    except Exception as e:
        return jsonify({"error": f"Error in branding application: {str(e)}"}), 500

    return jsonify({"image_url": final_image_url})

# Run the Flask app
if __name__ == '__main__':
    app.run(debug=True)
```

### Explanation of Backend

1. **Background Removal**: Receives the uploaded image, removes its background using DALL-E inpainting.
2. **Branding**: Adds a brand-specific background and color ring around the userâ€™s profile image.
3. **Response**: Returns the URL of the processed image.

---

## Frontend Code

Create a file named `index.html` for the frontend interface.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Brand Profile Image Generator</title>
    <style>
        #preview { width: 300px; border: 2px solid #ddd; border-radius: 50%; }
    </style>
</head>
<body>
    <h1>Brand Profile Image Generator</h1>
    
    <!-- Image Upload Form -->
    <form id="imageForm">
        <label for="image">Upload your image:</label>
        <input type="file" id="image" name="image" accept="image/*" required>
        
        <label for="brand_background">Choose Brand Background:</label>
        <select id="brand_background" name="brand_background">
            <option value="blue gradient">Blue Gradient</option>
            <option value="solid red">Solid Red</option>
        </select>
        
        <label for="brand_color">Choose Brand Color for Ring:</label>
        <input type="color" id="brand_color" name="brand_color" value="#ff0000">
        
        <button type="submit">Generate Profile Image</button>
    </form>

    <h2>Generated Profile Image</h2>
    <img id="preview" src="" alt="Your Branded Profile Image">

    <!-- JavaScript to handle form submission -->
    <script>
        document.getElementById('imageForm').addEventListener('submit', async (event) => {
            event.preventDefault();
            
            const formData = new FormData();
            const imageFile = document.getElementById('image').files[0];
            const brandBackground = document.getElementById('brand_background').value;
            const brandColor = document.getElementById('brand_color').value;
            
            formData.append('image', imageFile);
            formData.append('brand_background', brandBackground);
            formData.append('brand_color', brandColor);

            try {
                const response = await fetch('/process_image', {
                    method: 'POST',
                    body: formData
                });

                if (!response.ok) {
                    throw new Error("Error processing image");
                }

                const result = await response.json();
                const imageUrl = result.image_url;
                
                document.getElementById('preview').src = imageUrl;

            } catch (error) {
                alert("Failed to process image: " + error.message);
            }
        });
    </script>
</body>
</html>
```

### Explanation of Frontend

1. **Image Upload**: Allows user to upload an image, choose brand background and color.
2. **Form Submission**: Sends data to the Flask backend via AJAX.
3. **Image Display**: Shows the generated branded image.

---

## Running the Application

1. **Start the Flask Server**: Run the Flask backend:
    ```bash
    python app.py
    ```

2. **Open `index.html`** in a browser to access the frontend.

3. **Upload an Image** and select branding options to generate and view the final branded profile image.

---

## Summary

This application uses Flask to handle requests, OpenAI's DALL-E inpainting for background removal and branding, and HTML/JavaScript for the frontend interface. With this setup, users can easily create brand-customized profile images.

---

## Note

This setup requires internet access for API calls and an OpenAI API key for DALL-E inpainting.
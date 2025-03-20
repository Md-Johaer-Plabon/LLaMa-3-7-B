# Running LLaMa-3-7-B in colab

### **1️⃣ Install Dependencies and LLaMa environment in Colab cell**
📌Change Runtime type to T4 GPU or others.

Install dependencies:
```sh
!pip install flask pyngrok torch transformers sentencepiece
!pip install gdown
!pip install pyngrok
```

Install cmake to build LLaMa:
```sh
!apt-get update -y
!apt-get install -y git-lfs cmake build-essential python3-dev
```

Clone LLaMa environment:
```sha
!git clone https://github.com/ggerganov/llama.cpp.git
```

Build the environment to run LLaMa:
```sh
!cd llama.cpp && mkdir build && cd build && cmake .. && cmake --build . --parallel
```

### **2️⃣ Download LLaMa quantized model (llama-3-7b.Q4_K_M.gguf) from Hugging Face**
If you are a pro user, upload the "llama-3-7b.Q4_K_M.gguf" file directly in colab:
```python
from google.colab import files
uploaded = files.upload()
```

If you're not a pro user, upload the file in goole drive. You can access the file from any account.
Make the access of the uploaded file to "Anyone with the link".

Upload the file in colab from google drive:
```python
import gdown

# Replace the file id with your uploaded file
# Suppose this is your shared link: https://drive.google.com/file/d/1obx0ZqonvG8xT-SZZWebRW-ZNix1Yd7q/view?usp=sharing
# Then your file id is '1obx0ZqonvG8xT-SZZWebRW-ZNix1Yd7q'
# The final link will be 'https://drive.google.com/uc?export=download&id={1obx0ZqonvG8xT-SZZWebRW-ZNix1Yd7q}'

shared_link = 'https://drive.google.com/uc?export=download&id={your_file_id}'
gdown.download(shared_link, output='/content/Meta-Llama-3-7B-29Layers.IQ3_M.gguf', quiet=False)
```

### **3️⃣ Get response from LLaMa using Flask server**
📌As colab can't be access from your local pc, you have to publish the colab local server using ngrok to access from any local pc.

📌Go to the [sign up](https://dashboard.ngrok.com/signup) page of ngrok. Complete sign up and copy the auth token from dashboard.

Run the command in your colab cell:
```sh
!ngrok authtoken {YOUR_AUTH_TOKEN} #Replace it with your ngrok Auth Token
```

📌Create api.py file locally with the following python code to run flask server in colab:
 Upload the file manually in your 'content' folder.
```python
from flask import Flask, request, jsonify
from flask_cors import CORS
import subprocess
from pyngrok import ngrok

app = Flask(__name__)
CORS(app)

@app.route("/generate", methods=["POST"])
def generate_response():
    data = request.json
    prompt = data.get("prompt", "")

    # Run LLaMA 3 and get response
    result = subprocess.run(
        [
            "./llama.cpp/build/bin/main", 
            "-m", MODEL_PATH, 
            "-p", prompt
        ],
        capture_output=True,
        text=True
    )

    return jsonify({"response": result.stdout})

# Expose API using Ngrok
public_url = ngrok.connect(5000)
print("Public API URL:", public_url)

if __name__ == "__main__":
    app.run(port=5000)

```

📌Run the file in your colab cell:
```sh
!python3 /content/api.py
```


📌Get reponse from local pc:
```python
import requests

url = "http://xyz123.ngrok.io/generate"  # Replace with the actual Ngrok URL
payload = {"prompt": "Generate interview questions based on my resume."}

response = requests.post(url, json=payload)
print(response.json()["response"])
```

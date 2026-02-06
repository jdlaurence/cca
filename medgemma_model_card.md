MedGemma 1.5 model card
Note: This card describes MedGemma 1.5, which is only available as a 4B multimodal instruction-tuned variant. For information on MedGemma 1 variants, refer to the MedGemma 1 model card.

Model documentation: MedGemma

Resources:

Model on Google Cloud Model Garden: MedGemma

Models on Hugging Face: Collection

Concept applications built using MedGemma: Collection

GitHub repository

Tutorial notebooks

License: The use of MedGemma is governed by the Health AI Developer Foundations terms of use. MedGemma has not been evaluated or optimized for multi-turn applications.

MedGemma's training may make it more sensitive to the specific prompt used than Gemma 3.

When adapting MedGemma developer should consider the following:

License: The use of MedGemma is governed by the Health AI Developer Foundations terms of use.

Support channels

Author: Google

Model information
This section describes the specifications and recommended use of the MedGemma model.

Description
MedGemma is a collection of Gemma 3 variants that are trained for performance on medical text and image comprehension. Developers can use MedGemma to accelerate building healthcare-based AI applications.

MedGemma 1.5 4B is an updated version of the MedGemma 1 4B model.

MedGemma 1.5 4B expands support for several new medical imaging and data processing applications, including:

High-dimensional medical imaging: Interpretation of three-dimensional volume representations of Computed Tomography (CT) and Magnetic Resonance Imaging (MRI).
Whole-slide histopathology imaging (WSI): Simultaneous interpretation of multiple patches from a whole slide histopathology image as input.
Longitudinal medical imaging: Interpretation of chest X-rays in the context of prior images (e.g., comparing current versus historical scans).
Anatomical localization: Bounding box–based localization of anatomical features and findings in chest X-rays.
Medical document understanding: Extraction of structured data, such as values and units, from unstructured medical lab reports.
Electronic Health Record (EHR) understanding: Interpretation of text-based EHR data.
In addition to these new features, MedGemma 1.5 4B delivers improved accuracy on medical text reasoning and modest improvement on standard 2D image interpretation compared to MedGemma 1 4B.

MedGemma utilizes a SigLIP image encoder that has been specifically pre-trained on a variety of de-identified medical data, including chest X-rays, dermatology images, ophthalmology images, and histopathology slides. The LLM component is trained on a diverse set of medical data, including medical text, medical question-answer pairs, FHIR-based electronic health record data, 2D and 3D radiology images, histopathology images, ophthalmology images, dermatology images, and lab reports for document understanding.

MedGemma 1.5 4B has been evaluated on a range of clinically relevant benchmarks to illustrate its baseline performance. These evaluations are based on both open benchmark datasets and internally curated datasets. Developers are expected to fine-tune MedGemma for improved performance on their use case. Consult the Intended use section for more details.

MedGemma is optimized for medical applications that involve a text generation component. For medical image-based applications that do not involve text generation, such as data-efficient classification, zero-shot classification, or content-based or semantic image retrieval, the MedSigLIP image encoder is recommended. MedSigLIP is based on the same image encoder that powers MedGemma 1 and MedGemma 1.5.

How to use
The following are some example code snippets to help you quickly get started running the model locally on GPU.

Note: If you need to use the model at scale, we recommend creating a production version using Model Garden. Model Garden provides various deployment options and tutorial notebooks, including specialized server-side image processing options for efficiently handling large medical images: Whole Slide Digital Pathology (WSI) or volumetric scans (CT/MRI) stored in Cloud DICOM Store or Google Cloud Storage (GCS).

First, install the Transformers library. Gemma 3 is supported starting from transformers 4.50.0.

$ pip install -U transformers

Next, use either the pipeline wrapper or the transformer API directly to send a chest X-ray image and a question to the model.

Note that CT, MRI and whole-slide histopathology images require some pre-processing; see the CT and WSI notebook for examples.

Run model with the pipeline API

from transformers import pipeline
from PIL import Image
import requests
import torch

pipe = pipeline(
    "image-text-to-text",
    model="google/medgemma-1.5-4b-it",
    torch_dtype=torch.bfloat16,
    device="cuda",
)

# Image attribution: Stillwaterising, CC0, via Wikimedia Commons
image_url = "https://upload.wikimedia.org/wikipedia/commons/c/c8/Chest_Xray_PA_3-8-2010.png"
image = Image.open(requests.get(image_url, headers={"User-Agent": "example"}, stream=True).raw)

messages = [
    {
        "role": "user",
        "content": [
            {"type": "image", "image": image},
            {"type": "text", "text": "Describe this X-ray"}
        ]
    }
]

output = pipe(text=messages, max_new_tokens=2000)
print(output[0]["generated_text"][-1]["content"])

Run the model directly

# Make sure to install the accelerate library first via `pip install accelerate`
from transformers import AutoProcessor, AutoModelForImageTextToText
from PIL import Image
import requests
import torch

model_id = "google/medgemma-1.5-4b-it"

model = AutoModelForImageTextToText.from_pretrained(
    model_id,
    torch_dtype=torch.bfloat16,
    device_map="auto",
)
processor = AutoProcessor.from_pretrained(model_id)

# Image attribution: Stillwaterising, CC0, via Wikimedia Commons
image_url = "https://upload.wikimedia.org/wikipedia/commons/c/c8/Chest_Xray_PA_3-8-2010.png"
image = Image.open(requests.get(image_url, headers={"User-Agent": "example"}, stream=True).raw)

messages = [
    {
        "role": "user",
        "content": [
            {"type": "image", "image": image},
            {"type": "text", "text": "Describe this X-ray"}
        ]
    }
]

inputs = processor.apply_chat_template(
    messages, add_generation_prompt=True, tokenize=True,
    return_dict=True, return_tensors="pt"
).to(model.device, dtype=torch.bfloat16)

input_len = inputs["input_ids"].shape[-1]

with torch.inference_mode():
    generation = model.generate(**inputs, max_new_tokens=2000, do_sample=False)
    generation = generation[0][input_len:]

decoded = processor.decode(generation, skip_special_tokens=True)
print(decoded)

Examples
Refer to the growing collection of tutorial notebooks to see how to use or fine-tune MedGemma.

Model architecture overview
The MedGemma model is built based on Gemma 3 and uses the same decoder-only transformer architecture as Gemma 3. To read more about the architecture, consult the Gemma 3 model card.

Technical specifications
Model type: Decoder-only Transformer architecture, see the Gemma 3 Technical Report
Input modalities: Text, vision (multimodal)
Output modality: Text only
Attention mechanism: Grouped-query attention (GQA)
Context length: Supports long context, at least 128K tokens
Key publication: https://arxiv.org/abs/2507.05201
Model created: 4B multimodal: Jan 13, 2026
Model version: 4B multimodal: 1.5.0
Citation
When using this model, please cite: Sellergren et al. "MedGemma Technical Report." arXiv preprint arXiv:2507.05201 (2025).

@article{sellergren2025medgemma,
  title={MedGemma Technical Report},
  author={Sellergren, Andrew and Kazemzadeh, Sahar and Jaroensri, Tiam and Kiraly, Atilla and Traverse, Madeleine and Kohlberger, Timo and Xu, Shawn and Jamil, Fayaz and Hughes, Cían and Lau, Charles and others},
  journal={arXiv preprint arXiv:2507.05201},
  year={2025}
}

Inputs and outputs
Input:

Text string, such as a question or prompt
Images, normalized to 896 x 896 resolution and encoded to 256 tokens each
Total input length of 128K tokens
Output:

Generated text in response to the input, such as an answer to a question, analysis of image content, or a summary of a document
Total output length of 8192 tokens
Performance and evaluations
MedGemma was evaluated across a range of different multimodal classification, report generation, visual question answering, and text-based tasks.

Key performance metrics
Imaging evaluations
The multimodal performance of MedGemma 1.5 4B was evaluated across a range of benchmarks, focusing on radiology (2D, longitudinal 2D, and 3D), dermatology, histopathology, ophthalmology, document understanding, and multimodal clinical reasoning. See Data card for details of individual datasets.

We also list the previous results for MedGemma 1 4B and 27B (multimodal models only), as well as for Gemma 3 4B for comparison.

Task / Dataset	Metric	Gemma 3 4B	
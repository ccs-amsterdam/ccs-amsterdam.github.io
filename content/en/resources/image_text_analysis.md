---
title: Image and Text Analysis using Multi-modal Embeddings
date: 2024-03-22
author: Justin Chun-ting Ho
---

# Image and Text Analysis using Multi-modal Embeddings

Author: Justin Chun-ting Ho

Last Update: 22 Mar 2024

Description: Workshop materials for the Image and Text Topic Modelling using Multi-modal Embeddings. Designed to work with Google Colab. Check the full version on https://github.com/justinchuntingho/ImageTextAnalysisWorkshop


## The Workflow

![](https://maartengr.github.io/BERTopic/getting_started/multimodal/images_and_text.svg)

## Setting Up


```python
!pip install bertopic
import zipfile
import urllib
import numpy as np
import pandas as pd
from tqdm import tqdm
from PIL import Image
from sentence_transformers import SentenceTransformer, util
from bertopic import BERTopic
import base64
from io import BytesIO
from IPython.display import HTML
```

## Getting Data


```python
zip_path, _ = urllib.request.urlretrieve("https://github.com/justinchuntingho/ImageTextAnalysisWorkshop/raw/main/data.zip")
with zipfile.ZipFile(zip_path, "r") as f:
    f.extractall("./")
```


```python
df = pd.read_csv('data/data.csv')
```


```python
df
```

# Model Training

## True MultiModal


```python
def get_concat_h_multi_resize(im_list, resample=Image.BICUBIC):
    min_height = min(im.height for im in im_list)
    im_list_resize = [im.resize((int(im.width * min_height / im.height), min_height),resample=resample)
                      for im in im_list]
    total_width = sum(im.width for im in im_list_resize)
    dst = Image.new('RGB', (total_width, min_height))
    pos_x = 0
    for im in im_list_resize:
        dst.paste(im, (pos_x, 0))
        pos_x += im.width
    return dst

def get_concat_v_multi_resize(im_list, resample=Image.BICUBIC):
    min_width = min(im.width for im in im_list)
    im_list_resize = [im.resize((min_width, int(im.height * min_width / im.width)),resample=resample)
                      for im in im_list]
    total_height = sum(im.height for im in im_list_resize)
    dst = Image.new('RGB', (min_width, total_height))
    pos_y = 0
    for im in im_list_resize:
        dst.paste(im, (0, pos_y))
        pos_y += im.height
    return dst

def get_concat_tile_resize(im_list_2d, resample=Image.BICUBIC):
    im_list_v = [get_concat_h_multi_resize(im_list_h, resample=resample) for im_list_h in im_list_2d]
    return get_concat_v_multi_resize(im_list_v, resample=resample)

def get_top_imgs(topic):
    top_imgs = probs_df[topic].nlargest(9).index
    im1 = Image.open(df['image_path'][top_imgs[0]])
    im2 = Image.open(df['image_path'][top_imgs[1]])
    im3 = Image.open(df['image_path'][top_imgs[2]])
    im4 = Image.open(df['image_path'][top_imgs[3]])
    im5 = Image.open(df['image_path'][top_imgs[4]])
    im6 = Image.open(df['image_path'][top_imgs[5]])
    im7 = Image.open(df['image_path'][top_imgs[6]])
    im8 = Image.open(df['image_path'][top_imgs[7]])
    im9 = Image.open(df['image_path'][top_imgs[8]])
    return get_concat_tile_resize([[im1, im2, im3],
                                    [im4, im5, im6],
                                    [im7, im8, im9]])

def image_base64(im):
    with BytesIO() as buffer:
        im.resize((600,600)).save(buffer, 'jpeg')
        return base64.b64encode(buffer.getvalue()).decode()

def image_formatter(im):
    return f'<img src="data:image/jpeg;base64,{image_base64(im)}">'

def truncate_sentence(sentence, tokenizer):
    cur_sentence = sentence
    tokens = tokenizer.encode(cur_sentence)
    if len(tokens) > 77:
        truncated_tokens = tokens[1:76]
        cur_sentence = tokenizer.decode(truncated_tokens)
        return truncate_sentence(cur_sentence, tokenizer)
    else:
        return cur_sentence
```


```python
from bertopic.backend import MultiModalBackend
from transformers import CLIPTokenizer
model = MultiModalBackend('clip-ViT-B-32', batch_size=32)
tokenizer = CLIPTokenizer.from_pretrained("openai/clip-vit-large-patch14")

docs = [truncate_sentence(x,tokenizer) for x in df['text'].tolist()]
images = df['image_path'].tolist()
```


```python
docs[0:6]
```


```python
images[0:6]
```


```python
# Embed both images and documents, then average them
doc_image_embeddings = model.embed(docs, images)
```


```python
topic_model = BERTopic(calculate_probabilities=True,
                       n_gram_range=(1,2),
                       min_topic_size=5, # Setting this based on the smallest category in GS
                       verbose=True)

topics, probs = topic_model.fit_transform(docs, doc_image_embeddings)
```


```python
topic_model.get_topic_info()
```


```python
df['topic'] = topics
probs_df = pd.DataFrame(probs)
```


```python
# Extract dataframe
topic_info = topic_model.get_topic_info().drop("Representative_Docs", axis=1).drop("Name", axis=1).drop(index=0)
topic_info['Visual'] = [get_top_imgs(x) for x in topic_info.Topic]
```


```python
HTML(topic_info.to_html(formatters={'Visual': image_formatter}, escape=False,index=False))
```


```python
with open('multimodal.html', 'w') as fo:
    fo.write(topic_info.to_html(formatters={'Visual': image_formatter}, escape=False,index=False))
```

## Image Only


```python
def image_base64(im):
    if isinstance(im, str):
        im = get_thumbnail(im)
    with BytesIO() as buffer:
        im.save(buffer, 'jpeg')
        return base64.b64encode(buffer.getvalue()).decode()
```


```python
from bertopic.representation import KeyBERTInspired, VisualRepresentation
from bertopic.backend import MultiModalBackend

# Image embedding model
embedding_model = MultiModalBackend('clip-ViT-B-32', batch_size=32)

# Image to text representation model
representation_model = {
    "Visual_Aspect": VisualRepresentation(image_to_text_model="nlpconnect/vit-gpt2-image-captioning")
}
```


```python
# Train our model with images only
topic_model = BERTopic(embedding_model=embedding_model,
                       representation_model=representation_model,
                       min_topic_size=5,
                       calculate_probabilities=True)
topics, probs = topic_model.fit_transform(documents=None, images=df.image_path.to_list())
```


```python
df['topic'] = topics
probs_df = pd.DataFrame(probs)
```


```python
# Extract dataframe
topic_info = topic_model.get_topic_info().drop("Representative_Docs", axis=1).drop("Name", axis=1).drop(index=0)
```


```python
HTML(topic_info.to_html(formatters={'Visual_Aspect': image_formatter}, escape=False,index=False))
```


```python
with open('img.html', 'w') as fo:
    fo.write(topic_info.to_html(formatters={'Visual_Aspect': image_formatter}, escape=False, index=False))
```

## Covert to Text


```python
from transformers import pipeline
image_to_text = pipeline("image-to-text", model="nlpconnect/vit-gpt2-image-captioning")
```


```python
df['generated_text'] = [image_to_text(x)[0]['generated_text'] for x in df.image_path]
```


```python
docs = df.generated_text + df.text
```


```python
# Train our model with text only
topic_model = BERTopic(embedding_model=SentenceTransformer("all-MiniLM-L6-v2"),
                       n_gram_range=(1,2),
                       min_topic_size=5,
                       calculate_probabilities=True)
topics, probs = topic_model.fit_transform(documents=docs)
```


```python
df['topic'] = topics
probs_df = pd.DataFrame(probs)
```


```python
# Extract dataframe
topic_info = topic_model.get_topic_info().drop("Representative_Docs", axis=1).drop("Name", axis=1).drop(index=0)
```


```python
HTML(topic_info.to_html(escape=False,index=False))
```


```python
with open('text.html', 'w') as fo:
    fo.write(topic_info.to_html(escape=False,index=False))
```

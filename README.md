# How to grab key-value pairs from a local file
The purpose of these guides are to help users discover the potential uses of the data available via Lazarus Forms API. To get started, we will focus on how to get the Key-Value Pairs found on a document which was analyzed by the general model (https://api.lazarusforms.com/api/forms/generic)   

If you'd to follow along via a Jupyter Notebook, run `jupyter notebook` in this folder and open **Lazarus_Forms_Quickstart_Guide.ipynb**.

If you haven’t already, [create a Lazarus Forms account](https://lzr-forms-api-1-0.webflow.io/quickstart-guide) and grab your **authKey** and **orgId**.
___

The only package you will need to import is the requests package

```
import requests
```

For this example, we will be using an example form that is attached to this notebook, though you can select another image or pdf.

```
form_to_analyze = open('./Example Form.png', 'rb')
```

Once you have a pdf or image that you want to analyze, we can start making a POST request to the url below:

```
url = "https://api.lazarusforms.com/api/forms/generic"
```

As stated in the docs, we are going to need to pass the **authKey** and **orgId** in the headers.

**Make sure to add in your credentials here.**

```
headers = {
  'orgId': 'your orgID here',
  'authKey': 'your authKey here'
}
```

We will also have to pass **form_to_analyze** (as part of a multipart form).

```
files=[
  ('file',('file_name', form_to_analyze, 'application/octet-stream'))
]
```

Now we have all we need to run the POST request!

```
response = requests.post(url, headers=headers, files=files)
```

If all went well, you should have recieved a JSON response.

There is a chance you run into an error here so it is best to use a try-except here.

```
try:
    data_to_use = response.json()
except:
    print("There was an error")
    print(response)
```

While having a giant JSON is nice to look at, in this task, we are only interested in the 'keyValuePairs', which is an array.

```
key_values_pairs = data_to_use['keyValuePairs']
```

Now we need to parse through the pairs and grab the content of each key and value.

One thing to note is that some pairs may have a missing value if the model did not detect it or the form field was left blank, in this case we can avoid grabing that pair. 

```
key_values = []
for pair in key_values_pairs:
    try:
        key_values.append({
            'key' : pair['key']['content'],
            'value' : pair['value']['content']
        })
    except:
        pass
```

At this point, we now have a list of filled in form fields.

```
print(key_values)
```
___
As mentioned earlier, there may be a case where some of the fields were left blank or not detected, it would be useful to collect those incomplete key-values a present them to a reviewer to confirm and make corrections.

```
key_values = []
missing_values = []
for pair in key_values_pairs:
    try:
        key_values.append({
            'key' : pair['key']['content'],
            'value' : pair['value']['content']
        })
    except:
        print('Missing: ' + pair['key']['content'])
        missing_values.append({
            'key' : pair['key']['content']
        })
```
___
By the end of this, your code should look like this:
```
import requests

form_to_analyze = open('./Example Form.png', 'rb')
url = "https://api.lazarusforms.com/api/forms/generic"

headers = {
  'orgId': 'your orgID here',
  'authKey': 'your authKey here'
}

files=[
  ('file',('file_name', form_to_analyze, 'application/octet-stream'))
]

response = requests.post(url, headers=headers, files=files)

try:
    data_to_use = response.json()
except:
    print("There was an error")
    print(response)

key_values_pairs = data_to_use['keyValuePairs']

key_values = []
missing_values = []
for pair in key_values_pairs:
    try:
        key_values.append({
            'key' : pair['key']['content'],
            'value' : pair['value']['content']
        })
    except:
        print('Missing: ' + pair['key']['content'])
        missing_values.append({
            'key' : pair['key']['content']
        })
```

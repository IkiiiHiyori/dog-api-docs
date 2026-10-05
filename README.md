# dog-api-docs
# Dog API Documentation (Revised)

Public API for random dog images and breed data. No authentication required.

**API Version:** 1.0 (stable)
**Last updated:** May 2026

## Changelog

- v1.0 - Initial public documentation for stable endpoints.
- Documentation refresh - Added OpenAPI guidance, rate limit guidance, expanded error handling, client-side pagination, and retry examples.

**Base URL:** `https://dog.ceo/api`

### OpenAPI Specification

An OpenAPI (Swagger) specification is available for this API. You can use it to generate client libraries, import into Postman, or validate requests.

[Download openapi.yaml](./openapi.yaml)

Place `openapi.yaml` in the same directory as this documentation, or update the link to the location where the specification is hosted.

Minimal example:

```yaml
openapi: 3.0.3
info:
  title: Dog API
  version: "1.0"
servers:
  - url: https://dog.ceo/api
paths:
  /images/random:
    get:
      summary: Get a random dog image
      responses:
        "200":
          description: Random dog image returned
```

**Response format:** JSON

### Rate Limits

The API is free and publicly available. To keep the service reliable, clients should follow these limits and best practices:

- **Maximum requests per second:** 10 requests per second.
- **Caching:** Cache breed lists, image URLs, and full breed image collections where possible. Breed metadata changes infrequently, and image URLs can usually be reused.
- **Retry behavior:** If a response includes a `Retry-After` header, wait for the specified duration before making another request. If no `Retry-After` header is present, use exponential backoff before retrying.
- **Limit exceeded:** Exceeding the recommended limit may result in `429 Too Many Requests` responses. Clients should treat `429` as retryable after a delay.

```bash
curl https://dog.ceo/api/images/random
```

## Quick Start

Open a terminal or browser and run:

```bash
curl https://dog.ceo/api/images/random
```

The API returns a response like this:

```json
{
  "status": "success",
  "message": "https://images.dog.ceo/breeds/hound-afghan/n02088094_1003.jpg"
}
```

Copy the URL from the `"message"` field and open it in a browser to see the image.

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/breeds/list/all` | All breeds with sub-breeds |
| GET | `/images/random` | Random dog image |
| GET | `/breed/{breed}/images/random` | Random image of a specific breed |
| GET | `/breed/{breed}/images` | All images for a breed |
| GET | `/breed/{breed}/list` | Sub-breeds (if any) |
| GET | `/breed/{breed}/{subBreed}/images/random` | Random image of a sub-breed |

### 1. List All Breeds

Returns a complete list of all dog breeds available in the API, including sub-breeds. Use this to populate dropdowns or discover what's available.

**URL:** `/breeds/list/all`

```bash
curl https://dog.ceo/api/breeds/list/all
```

```javascript
fetch('https://dog.ceo/api/breeds/list/all')
  .then(res => res.json())
  .then(data => console.log(data.message));
```

Response:

```json
{
  "status": "success",
  "message": {
    "affenpinscher": [],
    "african": [],
    "airedale": [],
    "akita": [],
    "appenzeller": [],
    "australian": [
      "shepherd"
    ],
    "basenji": [],
    "beagle": [],
    "bluetick": [],
    "borzoi": [],
    "bouvier": [],
    "boxer": [],
    "brabancon": [],
    "briard": [],
    "buhund": [
      "norwegian"
    ],
    "bulldog": [
      "boston",
      "english",
      "french"
    ],
    "bullterrier": [
      "staffordshire"
    ],
    "corgi": [
      "cardigan"
    ],
    "hound": [
      "afghan",
      "basset",
      "blood",
      "english",
      "ibizan",
      "plott",
      "walker"
    ]
  }
}
```

This endpoint rarely errors, but if the API is unavailable:

```json
{
  "status": "error",
  "message": "Error fetching breeds"
}
```

### 2. Random Dog Image

Returns a single random dog image from any breed.

**URL:** `/images/random`

```bash
curl https://dog.ceo/api/images/random
```

```javascript
fetch('https://dog.ceo/api/images/random')
  .then(res => res.json())
  .then(data => {
    const imageUrl = data.message;
    console.log(imageUrl);
  });
```

Response:

```json
{
  "status": "success",
  "message": "https://images.dog.ceo/breeds/retriever-golden/n02099601_100.jpg"
}
```

Error:

```json
{
  "status": "error",
  "message": "Error fetching random image"
}
```

### 3. Random Image of Specific Breed

Returns a random image for a given breed. Valid breed values include: `beagle`, `boxer`, `bulldog`, `chihuahua`, `collie`, `corgi`, `dachshund`, `doberman`, `germanshepherd`, `goldenretriever`, `husky`, `labrador`, `poodle`, `pug`, `rottweiler`, `shiba`, `siberianhusky`, `yorkshireterrier`, and others.

**URL:** `/breed/{breed}/images/random`

```bash
curl https://dog.ceo/api/breed/beagle/images/random
```

```javascript
fetch('https://dog.ceo/api/breed/beagle/images/random')
  .then(res => res.json())
  .then(data => console.log(data.message));
```

Response:

```json
{
  "status": "success",
  "message": "https://images.dog.ceo/breeds/beagle/n02088364_10121.jpg"
}
```

Error:

```json
{
  "status": "error",
  "message": "Breed not found"
}
```

Breed names are case-sensitive and must match exactly what's returned by `/breeds/list/all`. Validate breed names before using this endpoint.

### 4. All Images for a Breed

Returns all available images for a specific breed. Useful for building a gallery or when you need to display several images from the same breed.

**URL:** `/breed/{breed}/images`

```bash
curl https://dog.ceo/api/breed/husky/images
```

```javascript
fetch('https://dog.ceo/api/breed/husky/images')
  .then(res => res.json())
  .then(data => {
    console.log(`Found ${data.message.length} images`);
    data.message.forEach(url => console.log(url));
  });
```

Response:

```json
{
  "status": "success",
  "message": [
    "https://images.dog.ceo/breeds/husky/n02110185_12748.jpg",
    "https://images.dog.ceo/breeds/husky/n02110185_12931.jpg",
    "https://images.dog.ceo/breeds/husky/n02110185_13133.jpg"
  ]
}
```

#### Pagination

The API does not natively support pagination for this endpoint. Some breeds have hundreds of images, so clients should apply pagination or display limits in the application layer.

Recommended client-side strategies:

- Use `Array.slice()` to limit the images displayed at one time.
- Fetch all images once and cache them for the current session or in your application cache.
- Implement lazy loading or infinite scroll with a fixed page size, such as 20 images per load.

Example client-side pagination:

```javascript
const pageSize = 20;
let currentPage = 0;
let cachedImages = [];

async function fetchBreedImages(breed) {
  if (cachedImages.length > 0) {
    return cachedImages;
  }

  const response = await fetch(`https://dog.ceo/api/breed/${breed}/images`);
  const data = await response.json();

  if (data.status !== 'success') {
    throw new Error(data.message);
  }

  cachedImages = data.message;
  return cachedImages;
}

function getImagePage(images, page, size) {
  const start = page * size;
  const end = start + size;
  return images.slice(start, end);
}

async function loadNextPage(breed) {
  const images = await fetchBreedImages(breed);
  const page = getImagePage(images, currentPage, pageSize);

  page.forEach(url => {
    const img = document.createElement('img');
    img.src = url;
    img.loading = 'lazy';
    document.getElementById('gallery').appendChild(img);
  });

  currentPage += 1;
}
```

### 5. List Sub-Breeds

Returns all sub-breeds for a given breed. Not all breeds have sub-breeds. Breeds that do include: `australian`, `bulldog`, `corgi`, `hound`, `poodle`, `retriever`, `shepherd`, `terrier`, and others.

**URL:** `/breed/{breed}/list`

```bash
curl https://dog.ceo/api/breed/hound/list
```

```javascript
fetch('https://dog.ceo/api/breed/hound/list')
  .then(res => res.json())
  .then(data => console.log(data.message));
```

Response:

```json
{
  "status": "success",
  "message": [
    "afghan",
    "basset",
    "blood",
    "english",
    "ibizan",
    "plott",
    "walker"
  ]
}
```

If a breed has no sub-breeds, the response is `{"status":"success","message":[]}`.

### 6. Random Image of Sub-Breed

Returns a random image for a specific sub-breed.

**URL:** `/breed/{breed}/{subBreed}/images/random`

- `{breed}`: `hound`, `bulldog`, `retriever`, `poodle`, etc.
- `{subBreed}`: depends on the breed (e.g., for `hound`: `afghan`, `basset`, `blood`, etc.)

```bash
curl https://dog.ceo/api/breed/hound/afghan/images/random
```

```javascript
fetch('https://dog.ceo/api/breed/hound/afghan/images/random')
  .then(res => res.json())
  .then(data => console.log(data.message));
```

Response:

```json
{
  "status": "success",
  "message": "https://images.dog.ceo/breeds/hound-afghan/n02088094_1003.jpg"
}
```

Error:

```json
{
  "status": "error",
  "message": "Breed not found"
}
```

### Working with Sub-Breeds

Many breeds have sub-breed variants. The URL structure differs depending on whether you're targeting a sub-breed.

Without sub-breeds:

```text
/breed/{breed}/images/random
```

Example: `/breed/beagle/images/random` -> random beagle image

With sub-breeds:

```text
/breed/{breed}/{subBreed}/images/random
```

Example: `/breed/hound/afghan/images/random` -> random Afghan hound image

To walk through a real example with Afghan hound, first confirm that `hound` has sub-breeds:

```bash
curl https://dog.ceo/api/breed/hound/list
```

```json
{
  "status": "success",
  "message": ["afghan", "basset", "blood", "english", "ibizan", "plott", "walker"]
}
```

Then request the sub-breed image:

```bash
curl https://dog.ceo/api/breed/hound/afghan/images/random
```

```json
{
  "status": "success",
  "message": "https://images.dog.ceo/breeds/hound-afghan/n02088094_1003.jpg"
}
```

Tip: Check the sub-breed list with `/breed/{breed}/list` before accessing a specific sub-breed.

## Error Handling

The API uses standard HTTP status codes alongside a JSON response body.

| HTTP Code | Meaning | When it happens |
|-----------|---------|-----------------|
| `200 OK` | OK | Request successful |
| `404 Not Found` | Not Found | Breed, sub-breed, or endpoint is invalid |
| `429 Too Many Requests` | Rate limit exceeded | The client sends more than the recommended 10 requests per second |
| `500 Internal Server Error` | Internal Error | Temporary API issue |
| `502 Bad Gateway` | Bad Gateway | Temporary upstream or gateway issue |

Clients should check both the HTTP status code and the `status` field in the JSON response before using the `message` value.

All error responses follow this shape:

```json
{
  "status": "error",
  "message": "Error description here"
}
```

Common errors:

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `"Breed not found"` | Invalid breed name | Check spelling against `/breeds/list/all` |
| `"Error fetching breeds"` | API temporarily down | Retry after a few minutes |
| `"Error fetching random image"` | API temporarily down | Retry after a few minutes |

Always check the `status` field before using the response:

```javascript
fetch('https://dog.ceo/api/breed/beagle/images/random')
  .then(res => res.json())
  .then(data => {
    if (data.status === 'success') {
      console.log('Image URL:', data.message);
    } else {
      console.error('Error:', data.message);
    }
  })
  .catch(error => {
    console.error('Network error:', error);
  });
```

Try it: https://dog.ceo/api/breed/beagle/images/random

## Code Examples

**JavaScript - get a random dog image:**

```javascript
fetch('https://dog.ceo/api/images/random')
  .then(res => res.json())
  .then(data => {
    const imageUrl = data.message;
    document.getElementById('dog-image').src = imageUrl;
  });
```

**JavaScript - list all breeds:**

```javascript
fetch('https://dog.ceo/api/breeds/list/all')
  .then(res => res.json())
  .then(data => {
    const breeds = Object.keys(data.message);
    console.log('Available breeds:', breeds);
  });
```

**JavaScript - get random image for a specific breed:**

```javascript
const breed = 'goldenretriever';
fetch(`https://dog.ceo/api/breed/${breed}/images/random`)
  .then(res => res.json())
  .then(data => {
    if (data.status === 'success') {
      console.log('Image URL:', data.message);
    } else {
      console.error('Error:', data.message);
    }
  });
```

**Python - get a random dog image:**

```python
import requests

response = requests.get('https://dog.ceo/api/images/random')
data = response.json()
print(data['message'])
```

**Python - list all breeds:**

```python
import requests

response = requests.get('https://dog.ceo/api/breeds/list/all')
data = response.json()
breeds = list(data['message'].keys())
print(f'Found {len(breeds)} breeds')
print(breeds[:10])  # Print first 10 breeds
```

**Python - get random image for a specific breed:**

```python
import requests

breed = 'husky'
response = requests.get(f'https://dog.ceo/api/breed/{breed}/images/random')
data = response.json()

if data['status'] == 'success':
    print(f'Random {breed} image:', data['message'])
else:
    print('Error:', data['message'])
```

**cURL - common requests:**

```bash
# Random dog image
curl https://dog.ceo/api/images/random

# List all breeds
curl https://dog.ceo/api/breeds/list/all

# Random beagle image
curl https://dog.ceo/api/breed/beagle/images/random

# All husky images
curl https://dog.ceo/api/breed/husky/images

# Hound sub-breeds
curl https://dog.ceo/api/breed/hound/list

# Random Afghan hound image
curl https://dog.ceo/api/breed/hound/afghan/images/random
```

**HTML - interactive button example:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Random Dog Generator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 50px;
    }
    button {
      padding: 15px 30px;
      font-size: 18px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #45a049;
    }
    #dog-image {
      max-width: 500px;
      margin-top: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    }
  </style>
</head>
<body>
  <h1>Random Dog Generator</h1>
  <button onclick="getRandomDog()">Show Random Dog</button>
  <br>
  <img id="dog-image" src="" alt="Dog image will appear here" style="display:none;">

  <script>
    function getRandomDog() {
      fetch('https://dog.ceo/api/images/random')
        .then(res => res.json())
        .then(data => {
          const img = document.getElementById('dog-image');
          img.src = data.message;
          img.style.display = 'block';
        })
        .catch(error => {
          console.error('Error:', error);
          alert('Failed to fetch dog image');
        });
    }
  </script>
</body>
</html>
```

## Best Practices

**Cache images locally.** Dog images don't change. Once you have an image URL, cache it in your application or database to reduce API calls.

```javascript
const cache = {};

async function getRandomDog() {
  if (cache['random']) {
    return cache['random'];
  }
  const response = await fetch('https://dog.ceo/api/images/random');
  const data = await response.json();
  cache['random'] = data.message;
  return data.message;
}
```

**Validate breed names before using user input.** When users can enter breed names, check them against the full list first:

```javascript
async function getBreedImage(userBreed) {
  const breedsRes = await fetch('https://dog.ceo/api/breeds/list/all');
  const breedsData = await breedsRes.json();
  const validBreeds = Object.keys(breedsData.message);

  if (!validBreeds.includes(userBreed.toLowerCase())) {
    throw new Error('Breed not found');
  }

  const imageRes = await fetch(`https://dog.ceo/api/breed/${userBreed}/images/random`);
  return await imageRes.json();
}
```

**No API key required.** The Dog API is public and free. Don't build authentication flows around it.

**Populate breed dropdowns from `/breeds/list/all`.** This ensures users only select valid breeds:

```javascript
fetch('https://dog.ceo/api/breeds/list/all')
  .then(res => res.json())
  .then(data => {
    const select = document.getElementById('breed-select');
    Object.keys(data.message).forEach(breed => {
      const option = document.createElement('option');
      option.value = breed;
      option.textContent = breed.charAt(0).toUpperCase() + breed.slice(1);
      select.appendChild(option);
    });
  });
```

**Implement request timeouts.** Network issues can occur; don't let requests hang indefinitely:

```javascript
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000); // 5 second timeout

fetch('https://dog.ceo/api/images/random', { signal: controller.signal })
  .then(res => res.json())
  .then(data => console.log(data.message))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.error('Request timed out');
    } else {
      console.error('Network error:', error);
    }
  })
  .finally(() => clearTimeout(timeoutId));
```

### Retry Logic

Because the API is free and maintained by one person, transient failures may occur. Retry temporary failures with exponential backoff, and stop retrying after a small number of attempts.

The following example retries failed requests up to 3 times with delays of 1 second, 2 seconds, and 4 seconds:

```javascript
function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);

      if (response.ok) {
        return await response.json();
      }

      const retryableStatusCodes = [429, 500, 502, 503, 504];
      if (!retryableStatusCodes.includes(response.status)) {
        throw new Error(`Request failed with status ${response.status}`);
      }

      throw new Error(`Temporary API error: ${response.status}`);
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }

      const delay = 1000 * 2 ** attempt;
      await wait(delay);
    }
  }
}

fetchWithRetry('https://dog.ceo/api/images/random')
  .then(data => console.log(data.message))
  .catch(error => console.error('Request failed after retries:', error));
```

## Limitations

**Read-only.** The API only supports GET requests. You cannot upload images, add breeds, modify data, or delete anything.

**Image sizes vary.** Some images are several MB. Resize images server-side before displaying, use lazy loading, and set CSS size constraints.

**No unique image guarantee.** Random endpoints can return the same image multiple times. To avoid duplicates, track previously seen URLs in your application, or use `/breed/{breed}/images` to fetch the full set and shuffle locally.

**No SLA.** The API is free and maintained by one person. There is no guaranteed uptime, no support agreement, and changes may happen without notice. Don't use this API in critical production systems without fallback mechanisms and error handling.

## FAQ

**How do I get a random Labrador image?**

Labradors are listed under the breed name `labrador`:

```bash
curl https://dog.ceo/api/breed/labrador/images/random
```

```javascript
fetch('https://dog.ceo/api/breed/labrador/images/random')
  .then(res => res.json())
  .then(data => console.log(data.message));
```

**What if a breed has sub-breeds?**

You need to specify the sub-breed. For example, `hound` has sub-breeds like `afghan`, `basset`, etc.:

```bash
# Get all hound sub-breeds
curl https://dog.ceo/api/breed/hound/list

# Get a random Afghan hound image
curl https://dog.ceo/api/breed/hound/afghan/images/random
```

Using `/breed/hound/images/random` without a sub-breed will return an error.

**Can I get multiple random images in one request?**

No. The API doesn't support batch random requests. Options:

- Make multiple requests in parallel.
- Use `/breed/{breed}/images` to get all images for a breed, then select randomly in your code.

```javascript
async function getMultipleRandomDogs(count) {
  const promises = [];
  for (let i = 0; i < count; i++) {
    promises.push(fetch('https://dog.ceo/api/images/random').then(r => r.json()));
  }
  const results = await Promise.all(promises);
  return results.map(r => r.message);
}

getMultipleRandomDogs(3).then(images => console.log(images));
```

**Is there a mobile SDK?**

No official SDK. Because the API follows standard REST conventions and returns JSON responses, any HTTP client works: URLSession or Alamofire on iOS, Retrofit or OkHttp on Android, `fetch` or axios in React Native, and the `http` package in Flutter.

**Why do I sometimes get the same image twice?**

The selection is random with no deduplication. Some breeds have smaller image pools. Track previously seen URLs in your application and skip repeats as needed.

**Can I use the images commercially?**

The API serves image URLs from third-party sources, primarily the Stanford Dogs Dataset. The API maintainer doesn't hold ownership of the images. For commercial use, verify the license of the original source and consider using your own licensed images in production.

**What happens if the API goes down?**

Implement error handling, cache images locally, and keep fallback images ready:

```javascript
async function getDogImageWithFallback() {
  try {
    const response = await fetch('https://dog.ceo/api/images/random');
    const data = await response.json();
    if (data.status === 'success') {
      return data.message;
    }
  } catch (error) {
    console.error('API error, using fallback');
  }
  return 'https://example.com/fallback-dog.jpg';
}
```

The API is maintained at [dog.ceo](https://dog.ceo). Check the website for the latest updates and status information.

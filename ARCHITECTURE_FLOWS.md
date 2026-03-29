# Architecture & Flow Diagrams

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           new-index.html                            │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │    │
│  │  │ Add Product  │  │Edit Product  │  │POS Board │  │    │
│  │  │ Modal        │  │Modal (NEW)   │  │          │  │    │
│  │  └──────────────┘  └──────────────┘  └──────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│                          ↓                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │    JavaScript Modules                               │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │    │
│  │  │ api.js   │  │products. │  │ orders.js        │  │    │
│  │  │          │  │ js (NEW) │  │                  │  │    │
│  │  │(FIXED)   │  │ FUNCTIONS│  │                  │  │    │
│  │  └──────────┘  └──────────┘  └──────────────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│                          ↓                                    │
│                   HTTP Requests                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │ Authorization: Bearer JWT_TOKEN                    │     │
│  │ Content-Type: application/json or multipart/form-data    │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│              API Gateway (Port 8080)                         │
│  Routes all requests to microservices                        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│            Product Service (Port 8081)                       │
│  ┌────────────────────────────────────────────────────┐     │
│  │ ProductController (UPDATED)                        │     │
│  │  POST   /products                                  │     │
│  │  GET    /products                                  │     │
│  │  POST   /products/{id}/image         ⭐ (FIXED)    │     │
│  │  PUT    /products/{id}               ✨ (NEW)     │     │
│  │  DELETE /products/{id}               ✨ (NEW)     │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                    │
│  ┌────────────────────────────────────────────────────┐     │
│  │ ProductService (UPDATED)                           │     │
│  │  create()                                          │     │
│  │  list()                                            │     │
│  │  uploadImage()                                     │     │
│  │  update()                           ✨ (NEW)      │     │
│  │  delete()                           ✨ (NEW)      │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                    │
│  ┌────────────────────────────────────────────────────┐     │
│  │ Storage (GCS)                                      │     │
│  │  put()   - Upload image                            │     │
│  │  get()   - Retrieve image URL                      │     │
│  └────────────────────────────────────────────────────┘     │
│                          ↓                                    │
│  ┌────────────────────────────────────────────────────┐     │
│  │ MongoDB                                            │     │
│  │  Product Collection                                │     │
│  │  {                                                 │     │
│  │    _id: "uuid",                                    │     │
│  │    name: "Orange Juice",                           │     │
│  │    price: 4.99,                                    │     │
│  │    imageUrl: "https://gcs/..."                     │     │
│  │  }                                                 │     │
│  └────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│            Google Cloud Storage (GCS)                        │
│  Stores product images with URL references                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Product Creation Flow (Updated)

```
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click "Add Product"                    │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Form Fields:                                             │
│  • Product Name (required)                              │
│  • Product Price (required)                             │
│  • Product Image (MANDATORY - marked with *)            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Select image from computer             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Frontend Validation:                                     │
│  ✓ Image selected?                                      │
│  ✓ File size < 5MB?                                     │
│  ✓ File type = image?                                   │
│  → Show preview                                          │
│  → Enable "Create Product" button                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click "Create Product"                │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ products.addProduct() executes:                          │
│  1. Validate form (name, price)                         │
│  2. Check image file selected ✓ MANDATORY               │
│  3. Show button text: "⏳ Creating product..."          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ API Call 1: POST /products                              │
│ {                                                        │
│   "name": "Orange Juice",                               │
│   "price": 4.99                                         │
│ }                                                        │
│                                                          │
│ Response: { "id": "product-123", ... }                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ products.addProduct() continues:                         │
│  4. Got product ID: "product-123"                       │
│  5. Show button text: "📸 Uploading image..."          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ API Call 2: POST /products/{id}/image  ⭐ (FIXED)       │
│ FormData:                                                │
│  - file: <binary image data>                            │
│                                                          │
│ Response: {                                             │
│   "id": "product-123",                                 │
│   "name": "Orange Juice",                               │
│   "price": 4.99,                                        │
│   "imageUrl": "https://gcs/.../product-123-file.jpg"  │
│ }                                                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Success! ✅                                              │
│  • Product saved to MongoDB                             │
│  • Image uploaded to GCS                                │
│  • Modal closes                                         │
│  • Products list reloaded                               │
│  • New product appears in grid with image               │
└─────────────────────────────────────────────────────────┘
```

---

## ✏️ Product Edit Flow (New)

```
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click ✏️ Edit button on product       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ products.openEditModal(productId) executes:             │
│  1. Find product from allProducts array                 │
│  2. Populate form with:                                 │
│     • Current product name                              │
│     • Current product price                             │
│     • Current product image (if exists)                │
│  3. Show image preview                                  │
│  4. Open modal                                          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Edit Product Modal opens with form pre-filled:          │
│  • Product Name: "Orange Juice" (editable)             │
│  • Product Price: 4.99 (editable)                      │
│  • Current Image: Shown as preview (optional change)    │
│  • Buttons: Cancel | Update | Delete                   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: User modifies form:                   │
│  • Change name to "Fresh Orange Juice"  ← Change 1      │
│  • Change price to 5.49                 ← Change 2      │
│  • Optionally: Select NEW image         ← Change 3      │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click "Update" button                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ products.updateProduct() executes:                       │
│  1. Get product ID, name, price from form               │
│  2. Check if new image selected                         │
│  3. Show button text: "⏳ Updating..."                  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ API Call 1: PUT /products/{id}  ✨ (NEW)                │
│ {                                                        │
│   "name": "Fresh Orange Juice",                         │
│   "price": 5.49                                         │
│ }                                                        │
│                                                          │
│ Response: Product updated in database                   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ If new image was selected:                              │
│  → Show button text: "📸 Uploading image..."           │
│                                                          │
│  API Call 2: POST /products/{id}/image                 │
│  FormData:                                              │
│   - file: <new image data>                              │
│                                                          │
│  Response: Image uploaded to GCS                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Success! ✅                                              │
│  • Product updated                                      │
│  • Image updated (if provided)                          │
│  • Modal closes                                         │
│  • Products list reloaded                               │
│  • Updated product appears in grid                      │
└─────────────────────────────────────────────────────────┘
```

---

## 🗑️ Product Delete Flow (New)

```
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click ✏️ Edit, then "Delete" button  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Browser shows confirmation dialog:                      │
│  "Are you sure you want to delete this product?         │
│   This action cannot be undone."                        │
│                                                          │
│  [OK] [Cancel]                                          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ USER INTERACTION: Click "OK" to confirm                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ products.deleteProduct() executes:                       │
│  1. Get product ID                                      │
│  2. Show button text: "⏳ Deleting..."                  │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ API Call: DELETE /products/{id}  ✨ (NEW)               │
│                                                          │
│ Response: 204 No Content (deleted successfully)         │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ Success! ✅                                              │
│  • Product deleted from database                        │
│  • Modal closes                                         │
│  • Products list reloaded                               │
│  • Product disappears from grid                         │
│  • Success message shown                                │
└─────────────────────────────────────────────────────────┘
```

---

## 📊 Image Upload Flow (Detailed)

```
Browser File Input
        ↓
[User selects image file]
        ↓
Frontend Validation Layer:
  ✓ File selected?
  ✓ File size ≤ 5MB?        → If NO → Error toast
  ✓ File type = image/*?    → If NO → Error toast
  ✓ File type in [jpg, png, gif, webp]?
        ↓ YES - All pass
Show Image Preview
Enable Create Button
        ↓
[User clicks "Create Product"]
        ↓
Create FormData:
  formData.append('file', fileObject)
        ↓
POST to /products/{id}/image
  Headers:
    Authorization: Bearer TOKEN
    (NO Content-Type header - FormData auto-sets multipart)
        ↓
Backend Processing:
  ← Receive multipart form data
  ← Extract 'file' from form
  ← Get filename, content-type, input stream
  ← Sanitize filename
  ← Create object name: "{productId}-{sanitizedFilename}"
  ← Upload to GCS using StorageClient
  ← Get public URL from GCS
  ← Save URL to Product.imageUrl
  ← Save Product to MongoDB
        ↓
Response with imageUrl:
  {
    "id": "product-123",
    "name": "Orange Juice",
    "price": 4.99,
    "imageUrl": "https://gcs-url-to-image.jpg"
  }
        ↓
Frontend Receives Response:
  ✓ Extract imageUrl from response
  ✓ Close modal
  ✓ Reload products list
  ✓ Render product with image
        ↓
Product Card Display:
  <img src="https://gcs-url-to-image.jpg" 
       style="width:100%; height:100%; object-fit:cover;">
```

---

## 🔐 Authentication Flow

```
┌─────────────┐
│ User Login  │
└─────────────┘
        ↓
POST /auth/login
{
  "email": "user@example.com",
  "password": "password123"
}
        ↓
Backend generates JWT token
        ↓
Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
        ↓
Frontend stores in localStorage:
localStorage.setItem('pos_token', token)
        ↓
api.js retrieves and adds to all requests:
headers['Authorization'] = `Bearer ${this.token}`
        ↓
All subsequent API calls include JWT in Authorization header
```

---

## 📈 Data Flow: Products Collection (MongoDB)

```
Before any product operations:
[Empty Collection]

After creating product with image:
{
  "_id": ObjectId("..."),
  "id": "product-123",
  "name": "Orange Juice",
  "price": 4.99,
  "imageUrl": "https://gcs-bucket/.../product-123-orange.jpg"
}

After editing product:
{
  "_id": ObjectId("..."),
  "id": "product-123",
  "name": "Fresh Orange Juice",  ← Updated
  "price": 5.49,                  ← Updated
  "imageUrl": "https://gcs-bucket/.../product-123-fresh-orange.jpg"  ← Updated
}

After deleting product:
[Removed from collection]
```

---

## 🎯 Key Changes Visualization

```
ENDPOINT CHANGES:
═════════════════════════════════════════════════════════════

❌ BEFORE (BROKEN):
POST /products/upload-image
   └─ Error: 404 Not Found
   └ Reason: Endpoint doesn't exist

✅ AFTER (FIXED):
POST /products/{id}/image
   └─ Works correctly
   └─ Uploads image to GCS
   └─ Returns imageUrl

✨ NEW ENDPOINTS:
PUT /products/{id}
   └─ Updates product name/price
   └─ Optional image update

DELETE /products/{id}
   └─ Deletes product
   └─ Requires confirmation
```

---

**Note:** All flows assume proper authentication (JWT token present) and valid input data.



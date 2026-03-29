# Quick Reference - Product Management APIs

## 🔧 API Endpoints

### Products Management

#### Create Product
```http
POST /products
Content-Type: application/json

{
  "name": "Orange Juice",
  "price": 4.99
}

Response 201:
{
  "id": "product-uuid",
  "name": "Orange Juice",
  "price": 4.99,
  "imageUrl": null
}
```

#### Get All Products
```http
GET /products

Response 200:
[
  {
    "id": "product-uuid",
    "name": "Orange Juice",
    "price": 4.99,
    "imageUrl": "https://gcs-bucket/product-uuid-filename.jpg"
  }
]
```

#### Upload Product Image ⭐ (FIXED)
```http
POST /products/{id}/image
Content-Type: multipart/form-data

file: <binary-image-file>

Response 200:
{
  "id": "product-uuid",
  "name": "Orange Juice",
  "price": 4.99,
  "imageUrl": "https://gcs-bucket/product-uuid-filename.jpg"
}
```

#### Update Product ✨ (NEW)
```http
PUT /products/{id}
Content-Type: application/json

{
  "name": "Fresh Orange Juice",
  "price": 5.99
}

Response 200:
{
  "id": "product-uuid",
  "name": "Fresh Orange Juice",
  "price": 5.99,
  "imageUrl": "https://gcs-bucket/product-uuid-filename.jpg"
}
```

#### Delete Product ✨ (NEW)
```http
DELETE /products/{id}

Response 204 No Content
```

---

## 📱 Frontend Functions

### JavaScript API (`api.js`)

```javascript
// Get all products
await api.getProducts()

// Create product
await api.createProduct(name, price, type = 'juice')

// Upload image ⭐ (FIXED ENDPOINT)
await api.uploadImage(productId, file)

// Update product ✨ (NEW)
await api.updateProduct(productId, name, price, imageUrl = null)

// Delete product ✨ (NEW)
await api.deleteProduct(productId)
```

### Product Management (`products.js`)

```javascript
// Load all products from API
await products.loadProducts()

// Render products in grid
products.renderProductsGrid()

// Open add product modal
products.openAddModal()

// Add new product ✨ (SIMPLIFIED - no type dropdown)
await products.addProduct()

// Open edit product modal ✨ (NEW)
products.openEditModal(productId)

// Update existing product ✨ (NEW)
await products.updateProduct()

// Delete product ✨ (NEW)
await products.deleteProduct()

// Close all modals
products.closeModals()
```

---

## 🎯 Frontend Form Fields

### Add Product Modal
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| productName | text | Yes | Product name |
| productPrice | number | Yes | Product price |
| productImage | file | **YES** | Image (JPG/PNG, max 5MB) |

### Edit Product Modal
| Field | Type | Required | Notes |
|-------|------|----------|-------|
| editProductId | hidden | - | Product ID (auto-filled) |
| editProductName | text | Yes | Product name |
| editProductPrice | number | Yes | Product price |
| editProductImage | file | No | Optional new image |

---

## 🐛 Error Handling

### Common HTTP Responses

| Status | Meaning | Action |
|--------|---------|--------|
| 200 OK | Success | Proceed |
| 201 Created | Resource created | Proceed |
| 204 No Content | Deleted successfully | Proceed |
| 400 Bad Request | Invalid input | Check form fields |
| 404 Not Found | Product not found | Reload page |
| 500 Server Error | Backend error | Check server logs |

### Frontend Error Messages

```javascript
// From api.js error handling
error.message        // Error message from backend
error.status         // HTTP status code
error.data           // Full error response object

// Toast notifications
app.showToast('Message', 'success')   // Green
app.showToast('Message', 'error')     // Red
app.showToast('Message', 'info')      // Blue
app.showToast('Message', 'warning')   // Orange
```

---

## 📤 Image Upload Flow

### Creating Product with Image

```
User Form
  ↓
Validate Form (name, price, image)
  ↓
Call api.createProduct()
  ↓
✅ Get product ID
  ↓
Call api.uploadImage(productId, file)
  ↓
✅ Image URL returned
  ↓
Product displayed with image
```

### Updating Product with New Image

```
Edit Form with Product Data
  ↓
User modifies name/price
  ↓
User optionally selects new image
  ↓
Call api.updateProduct()
  ↓
✅ Product updated
  ↓
If image selected:
  Call api.uploadImage()
  ↓
  ✅ Image uploaded
  ↓
Refresh product list
```

---

## 🔐 Headers

All requests include (auto-added by api.js):

```
Authorization: Bearer {JWT_TOKEN}
Content-Type: application/json (auto-set for JSON)
Content-Type: multipart/form-data (auto-set for file upload)
```

---

## 📊 FormData for Image Upload

```javascript
const formData = new FormData();
formData.append('file', fileObject);  // Binary file data

// Sent as multipart/form-data
// NOT as application/json
```

---

## ✅ Form Validation Rules

### Product Name
- Required
- Min length: 1 character
- Max length: 255 characters

### Product Price
- Required
- Must be > 0
- Supports decimals (e.g., 4.99)

### Product Image
- Required (for create, optional for update)
- Allowed types: JPG, PNG, GIF, WebP, BMP
- Max size: 5MB
- Dimensions: No restriction (backend handles)

---

## 🧪 Test with cURL

### Create Product
```bash
curl -X POST http://localhost:8080/products \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Mango Juice",
    "price": 4.99
  }'
```

### Upload Image
```bash
curl -X POST http://localhost:8080/products/PRODUCT_ID/image \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "file=@/path/to/image.jpg"
```

### Update Product
```bash
curl -X PUT http://localhost:8080/products/PRODUCT_ID \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Fresh Mango Juice",
    "price": 5.99
  }'
```

### Delete Product
```bash
curl -X DELETE http://localhost:8080/products/PRODUCT_ID \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 🔗 Related Files

| File | Purpose |
|------|---------|
| `frontend/js/api.js` | API communication |
| `frontend/js/products.js` | Product business logic |
| `frontend/new-index.html` | UI modals |
| `services/product-service/src/main/java/.../ProductController.java` | Backend endpoints |
| `services/product-service/src/main/java/.../ProductService.java` | Service logic |

---

## 📚 Documentation Files

- `README_FIXES.md` - Full summary of changes
- `FIXES_APPLIED.md` - Detailed change list
- `TESTING_GUIDE.md` - Test cases and scenarios
- `CODE_CHANGES_DETAIL.md` - Code diffs
- `README.md` - Backend service setup

---

## 🎓 Key Points to Remember

1. ⭐ **Image endpoint fixed:** `POST /products/{id}/image` (not `/products/upload-image`)
2. ✨ **Image mandatory:** Cannot create product without image
3. ✨ **Edit endpoint new:** `PUT /products/{id}` for updates
4. ✨ **Delete endpoint new:** `DELETE /products/{id}` for removal
5. 📸 **Two-step create:** Product created first, then image uploaded
6. 🔐 **Image validation:** Max 5MB, image types only
7. 🎯 **Product type simplified:** All products created as "juice" type

---

**Last Updated:** 2026-03-27  
**Version:** 1.0  
**Status:** Production Ready



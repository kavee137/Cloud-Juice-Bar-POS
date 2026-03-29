# Testing Guide - Product Management Features

## Quick Start

### Backend Prerequisites
1. Config Server running on `:8888`
2. Eureka running on `:8761`
3. API Gateway running on `:8080`
4. Product Service running on `:8081`

### Frontend
- Open `frontend/new-index.html` in browser
- Point to `http://localhost:8080` as API gateway

---

## Test Case 1: Create Product with Mandatory Image

**Steps:**
1. Navigate to Products page
2. Click "+ Add Product" button
3. Enter product name: "Mango Juice"
4. Enter price: "4.99"
5. **Notice: "Product Photo" field shows asterisk (*) = MANDATORY**
6. Try clicking "Create Product" without selecting image
   - ❌ Should show error: "Product image is mandatory"
7. Click on photo area to select image
8. Select a JPG/PNG file from your computer
9. Image preview should appear
10. Click "Create Product" button
    - Shows "⏳ Creating product..." then "📸 Uploading image..."
11. Should see "✅ Product and image created successfully!"
12. Product should appear in grid with image

---

## Test Case 2: Edit Product

**Steps:**
1. Find product in Products page grid
2. Click ✏️ (Edit) button
3. Edit Product modal opens with:
   - Current product name
   - Current product price
   - Current product image (if exists)
4. Change product name to "Fresh Mango Juice"
5. Change price to "5.49"
6. Optionally select new image
7. Click "Update" button
   - Shows "⏳ Updating..." then optionally "📸 Uploading image..."
8. Should see "✅ Product updated successfully!"
9. Product grid should refresh with new data

---

## Test Case 3: Delete Product

**Steps:**
1. Find product in Products page grid
2. Click ✏️ (Edit) button
3. In Edit Product modal, click "Delete" button
4. Browser should show confirmation dialog:
   - "Are you sure you want to delete this product? This action cannot be undone."
5. Click "OK" to confirm
   - Shows "⏳ Deleting..."
6. Should see "✅ Product deleted successfully!"
7. Product should disappear from grid

---

## Test Case 4: POS Dashboard with Images

**Steps:**
1. Navigate to POS Dashboard
2. Verify all products display with images
3. Search for a product using search bar
4. Click any product card to add to cart
5. Verify cart updates
6. Click "+ Add Item" in cart
7. Select products and quantities
8. Click "Place Order" button
9. Verify order is created successfully

---

## Test Case 5: Image Upload Error Handling

**Steps:**
1. Click "+ Add Product"
2. Try uploading a file > 5MB
   - Should show error: "❌ File size must be less than 5MB"
3. Try uploading a non-image file (TXT, PDF, etc.)
   - Should show error: "❌ Please select a valid image file"
4. Upload valid image
   - Should show success and preview

---

## Test Case 6: Form Validation

**Steps:**
1. Click "+ Add Product"
2. Leave all fields empty
3. Click "Create Product"
   - Should show error: "Please fill all fields correctly"
4. Enter only name, no price
5. Click "Create Product"
   - Should show error: "Please fill all fields correctly"
6. Enter negative price "-5"
7. Click "Create Product"
   - Should show error: "Please fill all fields correctly"
8. Enter valid name and price, but no image
   - Should show error: "⚠️ Product image is mandatory"

---

## Browser Console Checks

### Success Flow
When creating product with image, should see in console:
```
Creating product: Mango Juice
Product created with ID: [ID]
Uploading image for product: [ID] File: [filename]
Image uploaded successfully
```

### Error Flow
If upload fails, should see:
```
Error creating product: Error: [error message]
```

---

## API Endpoints Being Called

### Create Product
```
POST /products
{
  "name": "Mango Juice",
  "price": 4.99
}
Response: { "id": "...", "name": "Mango Juice", "price": 4.99, "imageUrl": null }
```

### Upload Image
```
POST /products/{id}/image
Headers: multipart/form-data
Body: file (binary)
Response: { "id": "...", "name": "...", "price": ..., "imageUrl": "https://..." }
```

### Update Product
```
PUT /products/{id}
{
  "name": "Fresh Mango Juice",
  "price": 5.49
}
Response: { "id": "...", "name": "Fresh Mango Juice", "price": 5.49, "imageUrl": "..." }
```

### Delete Product
```
DELETE /products/{id}
Response: 204 No Content
```

---

## Common Issues & Solutions

### Issue: "Error: NoResourceFoundException: No static resource products/upload-image"
- **Solution:** Backend was updated. Make sure product service is rebuilt and redeployed.
- **Check:** Verify endpoint is `POST /products/{id}/image` not `/products/upload-image`

### Issue: Image doesn't show after upload
- **Solution:** Image might still uploading to GCS. Wait a few seconds and refresh page.
- **Check:** Open browser DevTools → Network tab, verify image URL returns 200 OK

### Issue: Modal doesn't appear when clicking edit
- **Solution:** Check browser console for JavaScript errors
- **Check:** Verify `new-index.html` includes the edit modal HTML

### Issue: Delete button doesn't work
- **Solution:** Make sure confirmation dialog is handled properly
- **Check:** Click "OK" on confirmation dialog

### Issue: "Product not found" error
- **Solution:** Product might have been deleted by another user
- **Check:** Refresh products list
- **Check:** Verify backend database still has product

---

## Performance Notes

1. **Image Upload Time:** Depends on file size and GCS network
2. **Product Creation:** Creates product first, then uploads image (2 API calls)
3. **Product Update:** Updates info first, then image if provided (1-2 API calls)
4. **Product List Reload:** Fetches all products after create/update/delete

---

## Checklist for Full Feature Testing

- [ ] Product creation with mandatory image validation
- [ ] Product edit with optional image update
- [ ] Product deletion with confirmation
- [ ] Image appears in product cards
- [ ] Images display in POS dashboard
- [ ] Add to cart works with images
- [ ] Images persist after page refresh
- [ ] Error messages are user-friendly
- [ ] Modal forms validate input correctly
- [ ] File size validation (5MB limit)
- [ ] File type validation (images only)

---

## Browser Compatibility

Tested on:
- Chrome 120+
- Firefox 121+
- Safari 17+
- Edge 121+

All modern browsers with:
- `fetch()` API support
- `FormData` support
- `FileReader` API support



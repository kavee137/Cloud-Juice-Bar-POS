# Detailed Code Changes

## Backend Changes

### ProductController.java - Added Endpoints

```java
// ADDED: DeleteMapping import
import org.springframework.web.bind.annotation.DeleteMapping;

// ADDED: PutMapping import  
import org.springframework.web.bind.annotation.PutMapping;

// EXISTING: PostMapping endpoint for image upload
@PostMapping(value = "/products/{id}/image", consumes = "multipart/form-data")
public ProductResponse uploadImage(@PathVariable("id") String id, @RequestPart("file") MultipartFile file)
        throws IOException {
    Product updated = productService.uploadImage(
            id,
            file.getOriginalFilename(),
            file.getContentType(),
            file.getInputStream()
    );
    return toResponse(updated);
}

// NEW: PUT endpoint to update product
@PutMapping("/products/{id}")
public ProductResponse update(@PathVariable("id") String id, @Valid @RequestBody CreateProductRequest request) {
    Product updated = productService.update(id, request);
    return toResponse(updated);
}

// NEW: DELETE endpoint to delete product
@DeleteMapping("/products/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable("id") String id) {
    productService.delete(id);
}
```

---

### ProductService.java - Added Methods

```java
// EXISTING: uploadImage method (unchanged)

// NEW: Update product method
public Product update(String productId, CreateProductRequest request) {
    Product product = productRepository.findById(productId)
            .orElseThrow(() -> new NotFoundException("Product not found"));
    product.setName(request.name());
    product.setPrice(request.price());
    Product updated = productRepository.save(product);
    log.info("Updated product. productId={}", productId);
    return updated;
}

// NEW: Delete product method
public void delete(String productId) {
    Product product = productRepository.findById(productId)
            .orElseThrow(() -> new NotFoundException("Product not found"));
    productRepository.delete(product);
    log.info("Deleted product. productId={}", productId);
}
```

---

## Frontend Changes

### api.js - Fixed and Added Methods

```javascript
// CHANGED: uploadImage endpoint
// BEFORE:
// return this.request('/products/upload-image', {

// AFTER:
uploadImage(productId, file) {
    const formData = new FormData();
    formData.append('file', file);

    return this.request(`/products/${productId}/image`, {
        method: 'POST',
        body: formData,
    });
}

// NEW: updateProduct method
updateProduct(productId, name, price, imageUrl = null) {
    const body = { name, price };
    if (imageUrl) {
        body.imageUrl = imageUrl;
    }

    return this.request(`/products/${productId}`, {
        method: 'PUT',
        body,
    });
}

// NEW: deleteProduct method
deleteProduct(productId) {
    return this.request(`/products/${productId}`, {
        method: 'DELETE',
    });
}
```

---

### products.js - Major Changes

#### 1. Updated renderProductsGrid - Changed button from upload to edit

```javascript
// BEFORE:
<button class="btn btn-outline btn-sm" onclick="products.openUploadModal('${product.id}')">
    📸
</button>

// AFTER:
<button class="btn btn-outline btn-sm" onclick="products.openEditModal('${product.id}')" title="Edit">
    ✏️
</button>
```

#### 2. NEW: openEditModal Function

```javascript
openEditModal(productId) {
    const product = this.getProduct(productId);
    if (!product) {
        app.showToast('Product not found', 'error');
        return;
    }

    // Populate form with existing data
    document.getElementById('editProductId').value = productId;
    document.getElementById('editProductName').value = product.name;
    document.getElementById('editProductPrice').value = product.price;

    // Show image preview if exists
    const preview = document.getElementById('editProductImagePreview');
    const placeholder = document.getElementById('editProductImagePlaceholder');
    const previewImg = document.getElementById('editProductImagePreviewImg');
    const imageInfo = document.getElementById('editProductImageInfo');

    if (product.imageUrl) {
        previewImg.src = product.imageUrl;
        preview.style.display = 'flex';
        placeholder.style.display = 'none';
        imageInfo.textContent = '✅ Image: Uploaded';
    } else {
        preview.style.display = 'none';
        placeholder.style.display = 'flex';
        imageInfo.textContent = '⚠️ No image uploaded yet';
    }

    document.getElementById('editProductImage').value = '';

    // Show modal
    document.getElementById('editProductModal').classList.add('show');
}
```

#### 3. NEW: handleEditProductImageSelect Function

```javascript
handleEditProductImageSelect(event) {
    const file = event.target.files[0];
    const preview = document.getElementById('editProductImagePreview');
    const placeholder = document.getElementById('editProductImagePlaceholder');
    const previewImg = document.getElementById('editProductImagePreviewImg');
    const imageInfo = document.getElementById('editProductImageInfo');

    if (!file) {
        preview.style.display = 'none';
        placeholder.style.display = 'flex';
        imageInfo.textContent = '';
        return;
    }

    // Validate file size (5MB)
    const maxSize = 5 * 1024 * 1024;
    if (file.size > maxSize) {
        app.showToast('❌ File size must be less than 5MB', 'error');
        document.getElementById('editProductImage').value = '';
        preview.style.display = 'none';
        placeholder.style.display = 'flex';
        imageInfo.textContent = '';
        return;
    }

    // Validate file type
    if (!file.type.startsWith('image/')) {
        app.showToast('❌ Please select a valid image file', 'error');
        document.getElementById('editProductImage').value = '';
        preview.style.display = 'none';
        placeholder.style.display = 'flex';
        imageInfo.textContent = '';
        return;
    }

    // Show preview
    const reader = new FileReader();
    reader.onload = (e) => {
        previewImg.src = e.target.result;
        preview.style.display = 'flex';
        placeholder.style.display = 'none';
        imageInfo.textContent = `📸 New image selected: ${file.name}`;
        app.showToast(`✅ Image ready: ${file.name}`, 'success');
    };
    reader.readAsDataURL(file);
}
```

#### 4. NEW: updateProduct Function

```javascript
async updateProduct() {
    const productId = document.getElementById('editProductId').value;
    const name = document.getElementById('editProductName').value.trim();
    const price = parseFloat(document.getElementById('editProductPrice').value);
    const imageFile = document.getElementById('editProductImage').files[0];

    // Validation
    if (!name || !price || price <= 0) {
        app.showToast('Please fill all fields correctly', 'error');
        return;
    }

    try {
        const updateBtn = document.querySelector('#editProductModal .modal-footer .btn-primary');
        const originalText = updateBtn.innerHTML;
        updateBtn.disabled = true;

        // Update basic product info
        updateBtn.innerHTML = '⏳ Updating...';
        await api.updateProduct(productId, name, price);

        // Upload new image if selected
        if (imageFile) {
            updateBtn.innerHTML = '📸 Uploading image...';
            await api.uploadImage(productId, imageFile);
        }

        app.showToast('✅ Product updated successfully!', 'success');
        this.closeModals();
        await this.loadProducts();
        this.renderProductsGrid();
        this.renderPosProductsGrid();

        updateBtn.disabled = false;
        updateBtn.innerHTML = originalText;
    } catch (error) {
        console.error('Error updating product:', error);
        app.showToast(error.message || 'Failed to update product', 'error');
        const updateBtn = document.querySelector('#editProductModal .modal-footer .btn-primary');
        updateBtn.disabled = false;
        updateBtn.innerHTML = 'Update';
    }
}
```

#### 5. NEW: deleteProduct Function

```javascript
async deleteProduct() {
    const productId = document.getElementById('editProductId').value;
    if (!productId) {
        app.showToast('Product ID not found', 'error');
        return;
    }

    if (!confirm('Are you sure you want to delete this product? This action cannot be undone.')) {
        return;
    }

    try {
        const deleteBtn = document.querySelector('#editProductModal .modal-footer .btn-danger');
        const originalText = deleteBtn.innerHTML;
        deleteBtn.disabled = true;
        deleteBtn.innerHTML = '⏳ Deleting...';

        await api.deleteProduct(productId);

        app.showToast('✅ Product deleted successfully!', 'success');
        this.closeModals();
        await this.loadProducts();
        this.renderProductsGrid();
        this.renderPosProductsGrid();

        deleteBtn.disabled = false;
        deleteBtn.innerHTML = originalText;
    } catch (error) {
        console.error('Error deleting product:', error);
        app.showToast(error.message || 'Failed to delete product', 'error');
        const deleteBtn = document.querySelector('#editProductModal .modal-footer .btn-danger');
        deleteBtn.disabled = false;
        deleteBtn.innerHTML = 'Delete';
    }
}
```

#### 6. SIMPLIFIED: addProduct Function

```javascript
// CHANGED: Removed product type parameter and dropdown
async addProduct() {
    const name = document.getElementById('productName').value.trim();
    const price = parseFloat(document.getElementById('productPrice').value);
    // REMOVED: const type = document.getElementById('productType').value;
    const imageFile = document.getElementById('productImage').files[0];

    // Validation
    if (!name || !price || price <= 0) {
        app.showToast('Please fill all fields correctly', 'error');
        return;
    }

    // Validate image is required
    if (!imageFile) {
        app.showToast('⚠️ Product image is mandatory. Please select an image.', 'error');
        return;
    }

    // ... validation code ...

    try {
        const createBtn = document.querySelector('#addProductModal .modal-footer .btn-primary');
        const originalText = createBtn.innerHTML;
        createBtn.disabled = true;

        // Step 1: Create product first
        createBtn.innerHTML = '⏳ Creating product...';
        console.log('Creating product:', name);

        // CHANGED: Always use 'juice' type instead of dropdown
        const productResponse = await api.createProduct(name, price, 'juice');
        const productId = productResponse.id;
        console.log('Product created with ID:', productId);

        // Step 2: Upload image
        createBtn.innerHTML = '📸 Uploading image...';
        console.log('Uploading image for product:', productId, 'File:', imageFile.name);

        await api.uploadImage(productId, imageFile);
        console.log('Image uploaded successfully');

        app.showToast('✅ Product and image created successfully!', 'success');

        this.closeModals();
        await this.loadProducts();
        this.renderProductsGrid();
        this.renderPosProductsGrid();

        createBtn.disabled = false;
        createBtn.innerHTML = originalText;
    } catch (error) {
        console.error('Error creating product:', error);
        app.showToast(error.message || 'Failed to create product', 'error');
        const createBtn = document.querySelector('#addProductModal .modal-footer .btn-primary');
        createBtn.disabled = false;
        createBtn.innerHTML = 'Create Product';
    }
}
```

#### 7. UPDATED: clearAddForm Function

```javascript
// CHANGED: Removed productType
clearAddForm() {
    document.getElementById('productName').value = '';
    document.getElementById('productPrice').value = '';
    // REMOVED: document.getElementById('productType').value = 'juice';
    document.getElementById('productImage').value = '';
    
    // Reset image preview
    const preview = document.getElementById('productImagePreview');
    const placeholder = document.getElementById('productImagePlaceholder');
    if (preview && placeholder) {
        preview.style.display = 'none';
        placeholder.style.display = 'flex';
    }
}
```

---

### new-index.html - Added Edit Modal & Removed Dropdown

#### 1. REMOVED from Add Product Modal

```html
<!-- REMOVED: Product Type Dropdown -->
<!-- <div class="form-group">
    <label for="productType">Product Type</label>
    <select id="productType">
        <option value="juice">Juice</option>
        <option value="chips">Chips</option>
        <option value="water">Water</option>
    </select>
</div> -->
```

#### 2. ADDED: Edit Product Modal (Complete)

```html
<!-- Edit Product Modal -->
<div class="modal" id="editProductModal">
    <div class="modal-content">
        <div class="modal-header">
            <h3>Edit Product</h3>
            <button class="btn btn-icon" onclick="products.closeModals()">✕</button>
        </div>
        <div class="modal-body">
            <input type="hidden" id="editProductId">
            <div class="form-group">
                <label for="editProductName">Product Name</label>
                <input type="text" id="editProductName" placeholder="e.g., Orange Juice">
            </div>
            <div class="form-group">
                <label for="editProductPrice">Price ($)</label>
                <input type="number" id="editProductPrice" placeholder="0.00" step="0.01" min="0.01">
            </div>
            <div class="form-group">
                <label>Product Image</label>
                <div id="editProductImagePreview" style="display:none; width:100%; height:200px; border-radius:8px; overflow:hidden; margin-bottom:10px;">
                    <img id="editProductImagePreviewImg" src="" alt="preview" style="width:100%; height:100%; object-fit:cover;">
                </div>
                <p id="editProductImageInfo" style="font-size:12px; color:var(--text-muted); margin-bottom:10px;"></p>
                <div class="image-upload-area" id="editProductImageUploadArea" onclick="document.getElementById('editProductImage').click()">
                    <input type="file" id="editProductImage" accept="image/*" style="display:none" onchange="products.handleEditProductImageSelect(event)">
                    <div class="image-upload-content">
                        <div id="editProductImagePlaceholder">
                            <div style="font-size:32px; margin-bottom:8px;">📸</div>
                            <p style="font-size:12px; margin:0; color:var(--text-muted);">Click to change photo</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="modal-footer">
            <button class="btn btn-danger" onclick="products.deleteProduct()">Delete</button>
            <div>
                <button class="btn btn-outline" onclick="products.closeModals()">Cancel</button>
                <button class="btn btn-primary" onclick="products.updateProduct()">Update</button>
            </div>
        </div>
    </div>
</div>
```

---

## Summary of Changes

| Component | Type | Change | Impact |
|-----------|------|--------|--------|
| ProductController | Backend | Added PUT/DELETE endpoints | Enables product update/delete |
| ProductService | Backend | Added update/delete methods | Implements business logic |
| api.js | Frontend | Fixed upload endpoint, added update/delete | Correct API calls |
| products.js | Frontend | Added edit functions, simplified add flow | Better UX |
| new-index.html | Frontend | Added edit modal, removed type dropdown | Cleaner interface |

---

## Breaking Changes

None! All changes are backward compatible.

---

## Migration Path

1. Deploy backend changes (ProductController + ProductService)
2. Update frontend files (api.js, products.js, new-index.html)
3. Clear browser cache and refresh
4. Test all functionality



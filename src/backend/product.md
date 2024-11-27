# Product Module Documentation

## Overview
The **Product Module** is a core component of the e-commerce platform responsible for managing product-related operations such as importing, creating, and managing products. It allows administrators and producers to upload product data in bulk, configure product details, and manage various product types like digital and physical products. This module plays a crucial role in streamlining product management and ensures seamless product creation and updates.

---

## Feature 1: Import Products from CSV

### Logical Flow

1. **CSV File Upload**:
   - The user uploads a CSV file via the `POST /import-csv` endpoint. This CSV file contains the product data that needs to be imported into the platform.

2. **File Validation**:
   - The backend checks if the uploaded file is in the correct format (`text/csv`). If the file type is invalid, a `BadRequestException` is thrown.

3. **CSV Parsing**:
   - Once validated, the system parses the CSV file into an array of rows using the `parseCSV` function. Each row represents a product and its associated attributes.

4. **CSV Data Validation**:
   - After parsing, the data is validated using the `validateCSVData` function. This function ensures that each product has all the required fields, such as product ID, title, description, and pricing. If any required fields are missing or incorrect, an error is thrown and returned to the user.

5. **Grouping Products**:
   - The rows are grouped by product ID to efficiently process the creation of products. This ensures that all variations (like different SKUs) of the same product are handled together.

6. **Product Creation**:
   - The system processes each grouped product and calls the `transformToProductDTO` function to create a structured `CreateProductDTO`. This DTO contains all the necessary product information, including variants (SKUs), descriptions, and media files.

7. **Asynchronous Processing**:
   - The product creation process is done asynchronously. The backend doesn't wait for the product creation to finish but starts processing all the products in parallel for efficiency. Each product creation is handled by calling the `createProduct` method.

8. **Response**:
   - Once the file is uploaded and the processing starts, the backend responds with a success message indicating that the product import has started, including the total number of products to be created.
   
   - The user is immediately notified that the import process is ongoing, and the system will continue to process the products in the background.

9. **Error Handling**:
   - If there are any issues during the import or product creation process, the backend catches and logs the errors. These errors are returned as part of the response to inform the user about the failure reasons (such as invalid product data or missing required fields).

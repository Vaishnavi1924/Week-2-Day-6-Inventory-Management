# Week-2-Day-6-Inventory-Management

1. Create Inventory Page

Create:

frontend/src/pages/InventoryManagement.jsx

Add:

import { useEffect, useState } from "react";
import API from "../services/api";

function InventoryManagement() {
  const [products, setProducts] = useState([]);
  const [search, setSearch] = useState("");
  const [filter, setFilter] = useState("all");
  const [message, setMessage] = useState("");
  const [loading, setLoading] = useState(false);

  const fetchProducts = async () => {
    try {
      setLoading(true);

      const response = await API.get(
        "/products/my-products"
      );

      setProducts(response.data.products);
    } catch (error) {
      setMessage(
        error.response?.data?.message ||
        "Unable to load inventory"
      );
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchProducts();
  }, []);

  const updateStock = async (product, newStock) => {
    try {
      const stock = Number(newStock);

      if (stock < 0) {
        setMessage("Stock cannot be negative");
        return;
      }

      await API.put(
        `/products/${product._id}`,
        {
          stock
        }
      );

      setMessage(
        `${product.name} stock updated successfully`
      );

      fetchProducts();
    } catch (error) {
      setMessage(
        error.response?.data?.message ||
        "Unable to update stock"
      );
    }
  };

  const totalStock = products.reduce(
    (total, product) =>
      total + Number(product.stock || 0),
    0
  );

  const lowStockProducts = products.filter(
    (product) =>
      product.stock > 0 &&
      product.stock <= 5
  );

  const outOfStockProducts = products.filter(
    (product) =>
      product.stock === 0
  );

  const filteredProducts = products.filter(
    (product) => {
      const matchesSearch =
        product.name
          .toLowerCase()
          .includes(search.toLowerCase());

      if (!matchesSearch) {
        return false;
      }

      if (filter === "low") {
        return (
          product.stock > 0 &&
          product.stock <= 5
        );
      }

      if (filter === "out") {
        return product.stock === 0;
      }

      if (filter === "available") {
        return product.stock > 5;
      }

      return true;
    }
  );

  return (
    <div className="inventory-page">

      <h1>Inventory Management</h1>

      {message && (
        <p className="inventory-message">
          {message}
        </p>
      )}

      {/* Statistics */}

      <div className="inventory-stats">

        <div className="inventory-stat">
          <h3>Total Products</h3>
          <p>{products.length}</p>
        </div>

        <div className="inventory-stat">
          <h3>Total Stock</h3>
          <p>{totalStock}</p>
        </div>

        <div className="inventory-stat low">
          <h3>Low Stock</h3>
          <p>{lowStockProducts.length}</p>
        </div>

        <div className="inventory-stat out">
          <h3>Out of Stock</h3>
          <p>{outOfStockProducts.length}</p>
        </div>

      </div>

      {/* Search and Filter */}

      <div className="inventory-controls">

        <input
          type="text"
          placeholder="Search product..."
          value={search}
          onChange={(e) =>
            setSearch(e.target.value)
          }
        />

        <select
          value={filter}
          onChange={(e) =>
            setFilter(e.target.value)
          }
        >
          <option value="all">
            All Products
          </option>

          <option value="available">
            Available
          </option>

          <option value="low">
            Low Stock
          </option>

          <option value="out">
            Out of Stock
          </option>
        </select>

      </div>

      {/* Inventory Table */}

      {loading ? (

        <p>Loading inventory...</p>

      ) : (

        <div className="inventory-table-container">

          <table className="inventory-table">

            <thead>
              <tr>
                <th>Product</th>
                <th>Category</th>
                <th>Price</th>
                <th>Current Stock</th>
                <th>Status</th>
                <th>Update Stock</th>
              </tr>
            </thead>

            <tbody>

              {filteredProducts.length === 0 ? (

                <tr>
                  <td colSpan="6">
                    No products found
                  </td>
                </tr>

              ) : (

                filteredProducts.map(
                  (product) => {

                    let status = "In Stock";

                    if (product.stock === 0) {
                      status = "Out of Stock";
                    } else if (
                      product.stock <= 5
                    ) {
                      status = "Low Stock";
                    }

                    return (
                      <tr key={product._id}>

                        <td>
                          <strong>
                            {product.name}
                          </strong>
                        </td>

                        <td>
                          {product.category}
                        </td>

                        <td>
                          ₹{product.price}
                        </td>

                        <td>
                          {product.stock}
                        </td>

                        <td>

                          <span
                            className={
                              status === "In Stock"
                                ? "stock-status available"
                                : status === "Low Stock"
                                ? "stock-status low"
                                : "stock-status out"
                            }
                          >
                            {status}
                          </span>

                        </td>

                        <td>

                          <input
                            type="number"
                            min="0"
                            defaultValue={
                              product.stock
                            }
                            onBlur={(e) =>
                              updateStock(
                                product,
                                e.target.value
                              )
                            }
                          />

                        </td>

                      </tr>
                    );
                  }
                )

              )}

            </tbody>

          </table>

        </div>

      )}

    </div>
  );
}

export default InventoryManagement;
2. Add Inventory Route

Open:

frontend/src/App.jsx

Add the import:

import InventoryManagement from "./pages/InventoryManagement";

Then add this route:

<Route
  path="/inventory"
  element={
    <ProtectedRoute allowedRoles={["vendor"]}>
      <InventoryManagement />
    </ProtectedRoute>
  }
/>

Now the vendor can access:

http://localhost:5173/inventory
3. Add Inventory CSS

Open:

frontend/src/index.css

Add:

.inventory-page {
  max-width: 1200px;
  margin: 40px auto;
  padding: 20px;
}

.inventory-page h1 {
  margin-bottom: 25px;
}

.inventory-message {
  background: #e8f5e9;
  padding: 12px;
  border-radius: 6px;
  margin-bottom: 20px;
}

.inventory-stats {
  display: grid;
  grid-template-columns:
    repeat(4, 1fr);
  gap: 20px;
  margin-bottom: 30px;
}

.inventory-stat {
  background: white;
  padding: 25px;
  border-radius: 10px;
  text-align: center;
  box-shadow:
    0 2px 8px rgba(0, 0, 0, 0.1);
}

.inventory-stat h3 {
  margin-bottom: 10px;
}

.inventory-stat p {
  font-size: 30px;
  font-weight: bold;
}

.inventory-stat.low {
  border-left: 5px solid orange;
}

.inventory-stat.out {
  border-left: 5px solid red;
}

.inventory-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
}

.inventory-controls input,
.inventory-controls select {
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.inventory-controls input {
  flex: 1;
}

.inventory-table-container {
  background: white;
  border-radius: 10px;
  overflow-x: auto;
  box-shadow:
    0 2px 8px rgba(0, 0, 0, 0.1);
}

.inventory-table {
  width: 100%;
  border-collapse: collapse;
}

.inventory-table th,
.inventory-table td {
  padding: 15px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.inventory-table th {
  background: #f5f5f5;
}

.inventory-table input {
  width: 90px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.stock-status {
  display: inline-block;
  padding: 6px 10px;
  border-radius: 15px;
  font-size: 13px;
}

.stock-status.available {
  background: #e8f5e9;
  color: #2e7d32;
}

.stock-status.low {
  background: #fff3cd;
  color: #856404;
}

.stock-status.out {
  background: #ffebee;
  color: #c62828;
}

@media (max-width: 800px) {
  .inventory-stats {
    grid-template-columns:
      repeat(2, 1fr);
  }
}

@media (max-width: 500px) {
  .inventory-stats {
    grid-template-columns: 1fr;
  }

  .inventory-controls {
    flex-direction: column;
  }
}
4. Add Inventory Link to Vendor Dashboard

Open:

frontend/src/pages/VendorDashboard.jsx

Find:

<div className="dashboard-actions">

Add another link:

<Link to="/inventory">
  Inventory
</Link>

So it becomes:

<div className="dashboard-actions">

  <Link to="/products">
    Manage Products
  </Link>

  <Link to="/store-management">
    Manage Store
  </Link>

  <Link to="/inventory">
    Inventory
  </Link>

</div>
5. Test Inventory

Start your backend:

cd backend
node server.js

Start frontend:

cd frontend
npm run dev

Login as a vendor.

Go to:

http://localhost:5173/inventory

Suppose you have:

Product	Stock	Status
T-Shirt	20	In Stock
Shoes	4	Low Stock
Laptop Bag	0	Out of Stock

The dashboard will show:

Total Products       3
Total Stock         24
Low Stock             1
Out of Stock          1
6. Update Stock

Suppose:

Shoes
Current Stock: 4

Change the stock input to:

10

Click outside the input.

The frontend sends:

PUT /api/products/PRODUCT_ID

with:

{
  "stock": 10
}

Your existing product API updates only the stock field.

The status changes:

Low Stock
     ↓
In Stock
7. Important Stock Rules

We're using these simple rules:

Stock = 0
      ↓
Out of Stock


Stock = 1–5
      ↓
Low Stock


Stock > 5
      ↓
In Stock

Later, you can make the low-stock limit configurable for each store.

8. What You've Completed

Your vendor system now looks like:

                  VENDOR
                     │
                     ▼
             Vendor Dashboard
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
     Store         Products     Inventory
       │             │             │
       │         Add / Edit      Stock
       │         Delete          Status
       │         Images          Search
       │         Price           Filter
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                  MongoDB

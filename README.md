import React, { useState, useEffect } from "react";

// Cute Kawaii POS - Single-file React component
// Tailwind CSS assumed to be available in the project
// Uses localStorage for simple persistence

export default function CuteKawaiiPOS() {
  const THEME = {
    bg: "bg-[linear-gradient(180deg,#FFF7F1_0%,#FCF6F4_50%)]",
    card: "bg-[#f5e9e2]",
    accent: "#C7A98E", // pastel brown
    accentLight: "#EBDCCB",
    pink: "#F8D7E0",
    cream: "#FFF6EE",
  };

  const sampleProducts = [
    { id: 1, name: "Kakanin (Rice Cake)", price: 25.0, qty: 20 },
    { id: 2, name: "Sachet Shampoo", price: 10.0, qty: 50 },
    { id: 3, name: "Instant Coffee", price: 15.0, qty: 40 },
    { id: 4, name: "Chips (Small)", price: 12.0, qty: 30 },
  ];

  // products state with local persistence
  const [products, setProducts] = useState(() => {
    try {
      const raw = localStorage.getItem("kk_products");
      return raw ? JSON.parse(raw) : sampleProducts;
    } catch (e) {
      return sampleProducts;
    }
  });

  const [cart, setCart] = useState(() => {
    try {
      const raw = localStorage.getItem("kk_cart");
      return raw ? JSON.parse(raw) : [];
    } catch (e) {
      return [];
    }
  });

  const [query, setQuery] = useState("");
  const [showReceipt, setShowReceipt] = useState(false);
  const [lastSale, setLastSale] = useState(null);

  useEffect(() => {
    localStorage.setItem("kk_products", JSON.stringify(products));
  }, [products]);

  useEffect(() => {
    localStorage.setItem("kk_cart", JSON.stringify(cart));
  }, [cart]);

  // Basic cart helpers
  function addToCart(productId) {
    const product = products.find((p) => p.id === productId);
    if (!product || product.qty <= 0) return;
    setCart((c) => {
      const found = c.find((it) => it.id === productId);
      if (found) {
        return c.map((it) => (it.id === productId ? { ...it, qty: it.qty + 1 } : it));
      }
      return [...c, { id: product.id, name: product.name, price: product.price, qty: 1 }];
    });
  }

  function changeCartQty(productId, newQty) {
    if (newQty < 1) return removeFromCart(productId);
    setCart((c) => c.map((it) => (it.id === productId ? { ...it, qty: newQty } : it)));
  }

  function removeFromCart(productId) {
    setCart((c) => c.filter((it) => it.id !== productId));
  }

  function cartTotal() {
    return cart.reduce((s, it) => s + it.price * it.qty, 0);
  }

  function checkout(cash = null) {
    if (cart.length === 0) return alert("Cart is empty!");
    // update inventory
    setProducts((prev) =>
      prev.map((p) => {
        const inCart = cart.find((it) => it.id === p.id);
        if (!inCart) return p;
        return { ...p, qty: Math.max(0, p.qty - inCart.qty) };
      })
    );

    const sale = {
      id: Date.now(),
      items: cart,
      total: cartTotal(),
      paid: cash ?? cartTotal(),
      change: (cash ?? cartTotal()) - cartTotal(),
      date: new Date().toLocaleString(),
    };

    setLastSale(sale);
    setShowReceipt(true);
    setCart([]);
  }

  // product management
  function addProduct(payload) {
    setProducts((p) => [...p, { id: Date.now(), ...payload }]);
  }

  function updateProduct(id, payload) {
    setProducts((p) => p.map((it) => (it.id === id ? { ...it, ...payload } : it)));
  }

  function deleteProduct(id) {
    if (!confirm("Delete this product?")) return;
    setProducts((p) => p.filter((it) => it.id !== id));
  }

  // Simple Receipt component
  function Receipt({ sale, onClose }) {
    if (!sale) return null;
    return (
      <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
        <div className="w-full max-w-md rounded-2xl shadow-xl p-6" style={{ background: THEME.cream }}>
          <div className="flex items-center justify-between mb-4">
            <div>
              <h3 className="text-xl font-bold text-[#5b4636]">Kawaii Sari-Sari Store</h3>
              <p className="text-xs text-[#7b6b5e]">Receipt • {sale.date}</p>
            </div>
            <div className="text-right">
              <p className="text-sm font-semibold text-[#5b4636]">Total</p>
              <p className="text-2xl font-bold">₱{sale.total.toFixed(2)}</p>
            </div>
          </div>

          <div className="divide-y divide-[#e8dcd0]">
            {sale.items.map((it) => (
              <div key={it.id} className="py-2 flex justify-between items-center">
                <div>
                  <p className="font-medium text-[#5b4636]">{it.name}</p>
                  <p className="text-xs text-[#7b6b5e]">{it.qty} x ₱{it.price.toFixed(2)}</p>
                </div>
                <div className="font-semibold">₱{(it.qty * it.price).toFixed(2)}</div>
              </div>
            ))}
          </div>

          <div className="mt-4 flex justify-between text-sm text-[#5b4636]"><div>Paid</div><div>₱{sale.paid.toFixed(2)}</div></div>
          <div className="flex justify-between text-sm text-[#5b4636]"><div>Change</div><div>₱{sale.change.toFixed(2)}</div></div>

          <div className="mt-6 flex gap-3">
            <button onClick={onClose} className="flex-1 rounded-xl py-2 font-semibold shadow-sm text-white" style={{ background: THEME.accent }}>
              Close
            </button>
            <button
              onClick={() => window.print()}
              className="rounded-xl px-4 py-2 font-semibold border border-[#e6d7c8] bg-white text-[#5b4636]"
            >
              Print
            </button>
          </div>
        </div>
      </div>
    );
  }

  // Small Add Product form
  function AddProductForm() {
    const [name, setName] = useState("");
    const [price, setPrice] = useState(0);
    const [qty, setQty] = useState(1);

    function submit(e) {
      e.preventDefault();
      if (!name) return alert("Enter product name");
      addProduct({ name, price: Number(price), qty: Number(qty) });
      setName("");
      setPrice(0);
      setQty(1);
    }

    return (
      <form onSubmit={submit} className="p-3 rounded-xl" style={{ background: THEME.card }}>
        <h4 className="font-semibold text-[#5b4636] mb-2">Add product</h4>
        <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Product name" className="w-full mb-2 rounded-md px-3 py-2 text-sm outline-none" />
        <div className="flex gap-2">
          <input type="number" value={price} min="0" onChange={(e) => setPrice(e.target.value)} placeholder="Price" className="flex-1 rounded-md px-3 py-2 text-sm outline-none" />
          <input type="number" value={qty} min="0" onChange={(e) => setQty(e.target.value)} placeholder="Qty" className="w-24 rounded-md px-3 py-2 text-sm outline-none" />
        </div>
        <button type="submit" className="mt-2 w-full rounded-xl py-2 font-semibold text-white" style={{ background: THEME.accent }}>
          Add
        </button>
      </form>
    );
  }

  return (
    <div className={`min-h-screen p-6 ${THEME.bg}`}>
      <div className="max-w-6xl mx-auto grid grid-cols-12 gap-6">
        {/* Left: Products */}
        <div className="col-span-8">
          <header className="flex items-center justify-between mb-4">
            <div>
              <h1 className="text-3xl font-extrabold text-[#5b4636]">Kawaii Sari-Sari POS</h1>
              <p className="text-sm text-[#7b6b5e]">Pastel kawaii theme • simple and cute</p>
            </div>
            <div className="flex items-center gap-3">
              <div className="text-right">
                <p className="text-xs text-[#7b6b5e]">Inventory</p>
                <p className="text-lg font-semibold text-[#5b4636]">{products.reduce((s, p) => s + p.qty, 0)} items</p>
              </div>
            </div>
          </header>

          <div className="mb-3 flex items-center gap-3">
            <input value={query} onChange={(e) => setQuery(e.target.value)} placeholder="Search products..." className="flex-1 rounded-xl px-4 py-2 outline-none shadow-sm" />
            <div className="text-sm text-[#7b6b5e]">Theme: Kawaii Pastel</div>
          </div>

          <div className="grid grid-cols-3 gap-4">
            {products
              .filter((p) => p.name.toLowerCase().includes(query.toLowerCase()))
              .map((p) => (
                <div key={p.id} className="rounded-2xl p-4 shadow-sm" style={{ background: THEME.card }}>
                  <div className="flex items-start justify-between">
                    <div>
                      <h3 className="font-bold text-[#5b4636]">{p.name}</h3>
                      <p className="text-sm text-[#7b6b5e]">₱{p.price.toFixed(2)}</p>
                      <p className="text-xs mt-2 text-[#7b6b5e]">Stock: {p.qty}</p>
                    </div>
                    <div className="flex flex-col items-end gap-2">
                      <button onClick={() => addToCart(p.id)} className="rounded-full px-3 py-2 font-semibold text-white shadow" style={{ background: THEME.accent }}>
                        Add
                      </button>
                      <div className="text-xs text-[#7b6b5e]">ID: {p.id}</div>
                    </div>
                  </div>

                  <div className="mt-3 flex gap-2">
                    <button onClick={() => updateProduct(p.id, { qty: p.qty + 1 })} className="flex-1 rounded-lg py-2 text-sm border border-[#ecdacc]">+ Stock</button>
                    <button onClick={() => deleteProduct(p.id)} className="rounded-lg py-2 px-3 text-sm border border-[#f3dede]">Delete</button>
                  </div>
                </div>
              ))}
          </div>
        </div>

        {/* Right: Cart & Settings */}
        <aside className="col-span-4">
          <div className="sticky top-6 space-y-4">
            <div className="p-4 rounded-2xl shadow" style={{ background: THEME.card }}>
              <h4 className="font-semibold text-[#5b4636]">Cart</h4>

              <div className="mt-3 space-y-2">
                {cart.length === 0 && <p className="text-sm text-[#7b6b5e]">Cart is empty — add items from the left</p>}

                {cart.map((it) => (
                  <div key={it.id} className="flex items-center justify-between">
                    <div>
                      <p className="font-medium text-[#5b4636]">{it.name}</p>
                      <p className="text-xs text-[#7b6b5e]">₱{it.price.toFixed(2)} x {it.qty}</p>
                    </div>
                    <div className="flex items-center gap-2">
                      <button onClick={() => changeCartQty(it.id, it.qty - 1)} className="px-2 py-1 rounded bg-white">-</button>
                      <div className="px-2">{it.qty}</div>
                      <button onClick={() => changeCartQty(it.id, it.qty + 1)} className="px-2 py-1 rounded bg-white">+</button>
                    </div>
                  </div>
                ))}
              </div>

              <div className="mt-4 border-t pt-3 flex items-center justify-between">
                <div>
                  <p className="text-sm text-[#7b6b5e]">Total</p>
                  <p className="text-2xl font-bold text-[#5b4636]">₱{cartTotal().toFixed(2)}</p>
                </div>
                <div className="flex flex-col gap-2 w-36">
                  <button onClick={() => checkout()} className="rounded-xl py-2 font-semibold text-white" style={{ background: THEME.accent }}>
                    Checkout
                  </button>
                  <button onClick={() => { setCart([]); }} className="rounded-xl py-2 font-semibold border border-[#e6d7c8] bg-white text-[#5b4636]">Clear</button>
                </div>
              </div>
            </div>

            <AddProductForm />

            <div className="p-4 rounded-2xl shadow" style={{ background: THEME.card }}>
              <h4 className="font-semibold text-[#5b4636]">Quick Inventory</h4>
              <div className="mt-2 text-sm text-[#7b6b5e]">
                {products.map((p) => (
                  <div key={p.id} className="flex items-center justify-between py-1">
                    <div>{p.name}</div>
                    <div className="text-xs">{p.qty}</div>
                  </div>
                ))}
              </div>
            </div>

            <div className="p-4 rounded-2xl text-sm text-[#7b6b5e]" style={{ background: THEME.pink }}>
              <p className="font-semibold text-[#5b4636]">Tip</p>
              <p className="mt-1">Use the + Stock button to quickly add stock when a new delivery arrives. Use Print on receipt modal to give customer a paper receipt.</p>
            </div>
          </div>
        </aside>
      </div>

      {showReceipt && <Receipt sale={lastSale} onClose={() => setShowReceipt(false)} />}

      {/* Little floating credit */}
      <div className="fixed left-4 bottom-4 text-xs text-[#7b6b5e]">Made with 💕 • Cute Kawaii POS</div>
    </div>
  );
}

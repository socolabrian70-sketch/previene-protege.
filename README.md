# previene-protege.
previene-protege/
├─ index.html
├─ success.html
├─ cancel.html
├─ package.json
├─ vite.config.js
├─ tailwind.config.cjs
├─ postcss.config.cjs
├─ README.md
└─ src/
   ├─ main.jsx
   ├─ index.css
   ├─ App.jsx
   └─ components/
      └─ DualPurposeSite.jsx
└─ api/
   └─ create-checkout-session.js   ← Vercel Function (backend)
{
  "name": "previene-protege",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "tailwindcss": "^3.4.7",
    "postcss": "^8.4.21",
    "autoprefixer": "^10.4.14"
  }
}
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()]
})
<!doctype html>
<html lang="es">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Previene & Protege</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
<!doctype html>
<html>
  <head><meta charset="utf-8"/><title>Pago exitoso</title></head>
  <body>
    <h1>Pago completado</h1>
    <p>Gracias por tu compra. Tu pago se ha realizado con éxito.</p>
    <a href="/">Volver al sitio</a>
  </body>
</html>
<!doctype html>
<html>
  <head><meta charset="utf-8"/><title>Pago cancelado</title></head>
  <body>
    <h1>Pago cancelado</h1>
    <p>La transacción fue cancelada. Puedes intentar de nuevo.</p>
    <a href="/">Volver al sitio</a>
  </body>
</html>
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,jsx,ts,tsx}"
  ],
  theme: { extend: {} },
  plugins: [],
}
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
@tailwind base;
@tailwind components;
@tailwind utilities;

body { font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
import React from 'react'
import DualPurposeSite from './components/DualPurposeSite'

export default function App(){ return <DualPurposeSite /> }
import React, { useState } from 'react';

export default function DualPurposeSite() {
  const productsCatalog = [
    { id: 1, title: 'Pack Condones (10)', price: 15.0, desc: 'Pack de 10 condones de látex. Uso responsable.', image: 'https://images.unsplash.com/photo-1542362567-b07e54358753?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=8b9f6c2a6b8a4b6d' },
    { id: 2, title: 'Condón Ultra (1)', price: 3.0, desc: 'Condón de alta sensibilidad (unidad).', image: 'https://images.unsplash.com/photo-1583947582886-0f6d5d2c5b5e?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=3d6f7b9a2c1d5e6f' },
    { id: 3, title: 'Prueba de Embarazo (1)', price: 12.0, desc: 'Test rápido de embarazo de uso doméstico.', image: 'https://images.unsplash.com/photo-1582719478174-6a6a3b4f9d2b?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=5f4c3b2a1e9d8c7b' },
  ];

  const [cart, setCart] = useState([]);
  const [loadingCheckout, setLoadingCheckout] = useState(false);
  const [errorMsg, setErrorMsg] = useState('');

  const addToCart = (p) => setCart((c) => {
    const found = c.find(x => x.id === p.id);
    if (found) return c.map(x => x.id === p.id ? { ...x, qty: x.qty + 1 } : x);
    return [...c, { ...p, qty: 1 }];
  });

  const removeFromCart = (id) => setCart(c => c.filter(x => x.id !== id));
  const updateQty = (id, qty) => setCart(c => c.map(x => x.id===id ? { ...x, qty } : x));

  const total = () => cart.reduce((s, i) => s + i.price * i.qty, 0);

  async function handlePayNow() {
    if (cart.length === 0) { setErrorMsg('El carrito está vacío.'); return; }
    setErrorMsg(''); setLoadingCheckout(true);

    try {
      const resp = await fetch('/api/create-checkout-session', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          items: cart.map(({ id, title, price, qty }) => ({ id, title, price, qty })),
          success_url: window.location.origin + '/success',
          cancel_url: window.location.origin + '/cancel',
        }),
      });
      const data = await resp.json();
      if (data.url) { window.location = data.url; } else { setErrorMsg(data.error || 'No se pudo iniciar el pago.'); }
    } catch (err) {
      console.error(err); setErrorMsg('Error conectando con el servidor.');
    } finally { setLoadingCheckout(false); }
  }

  const formatCurrency = (value) => {
    try { return new Intl.NumberFormat('es-PE', { style: 'currency', currency: 'PEN' }).format(value); }
    catch (e) { return `S/ ${Number(value).toFixed(2)}`; }
  };

  return (
    <div className="min-h-screen bg-orange-50 text-slate-900">
      <header className="bg-white shadow">
        <div className="max-w-6xl mx-auto px-6 py-4 flex items-center justify-between">
          <div className="flex items-center gap-4">
            <div className="w-12 h-12 rounded-full bg-orange-500 flex items-center justify-center text-white font-bold">P</div>
            <div>
              <div className="text-lg font-extrabold">Previene & Protege</div>
              <div className="text-xs text-slate-500">Prevención del embarazo adolescente</div>
            </div>
          </div>
        </div>
      </header>

      <main className="max-w-6xl mx-auto px-6 py-10 grid grid-cols-1 lg:grid-cols-3 gap-8">
        <section className="lg:col-span-2 space-y-6">
          <div className="bg-white rounded-lg p-6 shadow-sm">
            <h1 className="text-2xl font-bold text-orange-700">Prevención del embarazo adolescente</h1>
            <p className="mt-2 text-slate-600">Información, recursos y charlas seleccionadas para jóvenes y educadores.</p>
          </div>

          <div className="grid grid-cols-1 sm:grid-cols-3 gap-4">
            <div className="bg-white rounded-lg shadow-sm overflow-hidden">
              <div className="aspect-video"><iframe className="w-full h-full" title="Charla 1" src="https://www.youtube.com/embed/MIxyV5Ot-ZU?start=7" frameBorder="0" allowFullScreen></iframe></div>
              <div className="p-4"><div className="font-medium">Charla 1 — Prevención y educación</div></div>
            </div>
            <div className="bg-white rounded-lg shadow-sm overflow-hidden">
              <div className="aspect-video"><iframe className="w-full h-full" title="Charla 2" src="https://www.youtube.com/embed/zn5iDGjJE_I" frameBorder="0" allowFullScreen></iframe></div>
              <div className="p-4"><div className="font-medium">Charla 2 — Información y testimonios</div></div>
            </div>
            <div className="bg-white rounded-lg shadow-sm overflow-hidden">
              <div className="aspect-video"><iframe className="w-full h-full" title="Charla 3" src="https://www.youtube.com/embed/LxndFZoOQA4" frameBorder="0" allowFullScreen></iframe></div>
              <div className="p-4"><div className="font-medium">Charla 3 — Prevención y recursos</div></div>
            </div>
          </div>

          <div className="bg-white rounded-lg p-6 shadow-sm">
            <h2 className="text-xl font-semibold">Tienda — Productos para prevención</h2>
            <p className="text-sm text-slate-600 mt-1">Compra discreta y segura. Los pagos se realizan con tarjeta a través de Stripe.</p>
          </div>

          <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
            {productsCatalog.map(p => (
              <div key={p.id} className="bg-white rounded-lg shadow-md overflow-hidden flex flex-col">
                <img src={p.image} alt={p.title} className="w-full h-44 object-cover" />
                <div className="p-4 flex-1 flex flex-col justify-between">
                  <div>
                    <div className="font-semibold">{p.title}</div>
                    <div className="text-sm text-slate-600 mt-1">{p.desc}</div>
                  </div>
                  <div className="mt-4 flex items-center justify-between">
                    <div className="text-lg font-bold text-orange-700">{formatCurrency(p.price)}</div>
                    <button onClick={() => addToCart(p)} className="bg-orange-600 text-white px-3 py-2 rounded">Añadir al carrito</button>
                  </div>
                </div>
              </div>
            ))}
          </div>
        </section>

        <aside>
          <div className="bg-white rounded-lg p-4 shadow-sm">
            <h4 className="font-semibold">Carrito</h4>
            {cart.length === 0 ? (
              <p className="text-slate-500 mt-2">Tu carrito está vacío.</p>
            ) : (
              <div className="space-y-3 mt-2">
                {cart.map(it => (
                  <div key={it.id} className="flex items-center justify-between">
                    <div>
                      <div className="font-medium">{it.title}</div>
                      <div className="text-sm text-slate-500">{formatCurrency(it.price)} x {it.qty}</div>
                    </div>
                    <div className="flex items-center gap-2">
                      <input type="number" min={1} value={it.qty} onChange={(e) => updateQty(it.id, Math.max(1, Number(e.target.value)))} className="w-16 border rounded px-2 py-1" />
                      <button onClick={() => removeFromCart(it.id)} className="text-red-600">Eliminar</button>
                    </div>
                  </div>
                ))}

                <div className="pt-2 border-t flex items-center justify-between font-medium">Total <span>{formatCurrency(total())}</span></div>

                {errorMsg && <div className="text-sm text-red-600 mt-2">{errorMsg}</div>}

                <button onClick={handlePayNow} disabled={loadingCheckout} className="mt-3 w-full bg-orange-600 text-white py-2 rounded">{loadingCheckout ? 'Redirigiendo al pago...' : 'Pagar ahora con tarjeta'}</button>

                <div className="text-xs text-slate-500 mt-3">Pago seguro procesado por Stripe. No compartimos tus datos de tarjeta con terceros.</div>
              </div>
            )}
          </div>

          <div className="bg-white rounded-lg p-4 shadow-sm mt-4">
            <h4 className="font-semibold">Recursos</h4>
            <ul className="mt-2 text-sm text-slate-600">
              <li><a href="#" className="text-orange-600 hover:underline">MINSA — Salud Sexual</a></li>
              <li className="mt-1"><a href="#" className="text-orange-600 hover:underline">UNICEF Perú — Jóvenes</a></li>
              <li className="mt-1"><a href="#" className="text-orange-600 hover:underline">OPS / OMS — Adolescentes</a></li>
            </ul>
          </div>
        </aside>
      </main>

      <footer className="bg-slate-900 text-slate-200 py-6">
        <div className="max-w-6xl mx-auto px-6 flex items-center justify-between">
          <div className="text-sm">© {new Date().getFullYear()} Previene & Protege</div>
          <div className="text-sm">Diseño: paleta naranja / beige</div>
        </div>
      </footer>
    </div>
  );
}
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    res.status(405).json({ error: 'Method not allowed' });
    return;
  }

  try {
    const { items, success_url, cancel_url } = req.body;

    const line_items = (items || []).map(it => ({
      price_data: {
        currency: 'pen',
        product_data: { name: it.title },
        unit_amount: Math.round(it.price * 100),
      },
      quantity: it.qty,
    }));

    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      mode: 'payment',
      line_items,
      success_url: success_url || `${req.headers.origin}/success`,
      cancel_url: cancel_url || `${req.headers.origin}/cancel`,
    });

    res.status(200).json({ url: session.url });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: err.message });
  }
}

/**
 * index.js
 * POST /sync-qikink  -> Manual sync from Qikink -> Firestore
 *
 * Environment variables expected:
 * - PORT (optional)
 * - QIK_CLIENT_ID
 * - QIK_CLIENT_SECRET
 * - QIK_TOKEN_URL (optional; default provided)
 * - QIK_PRODUCTS_URL (optional; default provided)
 * - FIREBASE_SERVICE_ACCOUNT (JSON string) OR GOOGLE_APPLICATION_CREDENTIALS (path) 
 *
 * Note: store all secrets in Render environment variables.
 */

const express = require('express');
const axios = require('axios');
const admin = require('firebase-admin');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());

// Config: set sensible defaults, but allow override with env
const PORT = process.env.PORT || 3000;
const QIK_CLIENT_ID = process.env.QIK_CLIENT_ID;
const QIK_CLIENT_SECRET = process.env.QIK_CLIENT_SECRET;
const QIK_TOKEN_URL = process.env.QIK_TOKEN_URL || 'https://api.qik.dev/authenticate';
const QIK_PRODUCTS_URL = process.env.QIK_PRODUCTS_URL || 'https://api.qik.dev/v1/products'; // configurable

if (!QIK_CLIENT_ID || !QIK_CLIENT_SECRET) {
  console.warn('QIK client id/secret not set. Set QIK_CLIENT_ID and QIK_CLIENT_SECRET in env.');
}

// Initialize Firebase Admin
function initFirebase() {
  // Two supported options:
  // 1) FIREBASE_SERVICE_ACCOUNT contains the raw JSON string of service account
  // 2) GOOGLE_APPLICATION_CREDENTIALS env points to a file path (Render: prefer option 1)
  if (!admin.apps.length) {
    if (process.env.FIREBASE_SERVICE_ACCOUNT) {
      let sa;
      try {
        sa = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT);
      } catch (e) {
        // Maybe it is base64-encoded
        try {
          sa = JSON.parse(Buffer.from(process.env.FIREBASE_SERVICE_ACCOUNT, 'base64').toString('utf8'));
        } catch (err) {
          console.error('FIREBASE_SERVICE_ACCOUNT parse error:', err.message);
          throw new Error('Invalid FIREBASE_SERVICE_ACCOUNT (must be JSON string or base64 JSON).');
        }
      }
      admin.initializeApp({
        credential: admin.credential.cert(sa)
      });
    } else {
      // fallback to default credentials (if GOOGLE_APPLICATION_CREDENTIALS is set)
      admin.initializeApp();
    }
  }
}
initFirebase();

const db = admin.firestore();

/**
 * Authenticate to Qikink sandbox and return access token.
 * Some docs indicate token should be passed as `authorization` header
 * without "Bearer" prefix. This code returns whatever token string we get.
 */
async function getQikToken() {
  // Using client id/secret in body; adapt if API requires basic auth or other shape.
  const payload = {
    client_id: QIK_CLIENT_ID,
    client_secret: QIK_CLIENT_SECRET
  };

  try {
    const res = await axios.post(QIK_TOKEN_URL, payload, {
      headers: { 'Content-Type': 'application/json' },
      timeout: 15000
    });
    // Expect token in res.data.access_token or res.data.token; be flexible:
    const token = res.data && (res.data.access_token || res.data.token || res.data.data?.token);
    if (!token) {
      console.warn('Token missing in Qik response:', res.data);
      throw new Error('No token received from Qikink authenticate endpoint.');
    }
    return token;
  } catch (err) {
    console.error('Error getting Qik token:', err.response?.data || err.message);
    throw err;
  }
}

/**
 * Fetch products from Qikink API using token.
 * Adjust query params / pagination as required.
 */
async function fetchQikProducts(token) {
  try {
    const headers = {
      authorization: token // NOTE: some docs say no "Bearer " prefix for sandbox tokens
    };
    const res = await axios.get(QIK_PRODUCTS_URL, { headers, timeout: 30000 });
    // Response may be {data: [...] } or { products: [...] } - try multiple locations
    const items = res.data && (res.data.data || res.data.products || res.data);
    if (!items || !Array.isArray(items)) {
      console.warn('Unexpected products payload shape:', res.data);
      throw new Error('Products response is not an array.');
    }
    return items;
  } catch (err) {
    console.error('Error fetching Qik products:', err.response?.data || err.message);
    throw err;
  }
}

/**
 * Map a single Qik product object to our Firestore document schema.
 * Adjust the mapping depending on actual Qik response keys.
 */
function mapProductToDoc(q) {
  // try common fallbacks
  const productId = q.id || q.ProductID || q['Product ID'] || q.product_id || q.sku || q.SKU;
  const name = q.name || q.title || q.Product || q.product_name || (q.product && q.product.name);
  const design = q.design || q.Design;
  const sku = q.sku || q.SKU || q.sku_code;
  const type = q.product_type || q.type || q['Product Type'] || q.category || 'uncategorized';
  const price = Number(q.price || q['Product Price (Starts from)'] || q.price_start || q.price_from) || 0;
  let image = '';
  // image can be array or string
  if (Array.isArray(q.images) && q.images.length) image = q.images[0];
  else if (q.image) image = q.image;
  else if (q.Image) image = q.Image;

  return {
    productId: String(productId || (q.sku || Date.now())),
    name: name || '',
    design: design || '',
    sku: sku || '',
    type: String(type).trim(),
    price,
    image: image || '',
    raw: q // store raw provider payload for debugging / future use
  };
}

/**
 * Upsert product doc into Firestore under collection = sanitized product type
 */
async function upsertProductToFirestore(prod) {
  const collectionName = sanitizeCollectionName(prod.type || 'uncategorized');
  const docId = prod.productId;
  const docRef = db.collection(collectionName).doc(docId);
  const now = admin.firestore.FieldValue.serverTimestamp();
  const dataToSave = {
    name: prod.name,
    design: prod.design,
    sku: prod.sku,
    type: prod.type,
    price: prod.price,
    image: prod.image,
    updatedAt: now,
    createdAt: now,
    providerRaw: prod.raw
  };
  // If doc exists, merge to avoid overwriting createdAt
  await docRef.set(dataToSave, { merge: true });
}

/** sanitize collection names (Firestore allows most chars but avoid slashes and leading dots) */
function sanitizeCollectionName(name) {
  return String(name || 'uncategorized').replace(/[\/\#\$\[\]\.]/g, '-').trim() || 'uncategorized';
}

/**
 * Main sync route (manual trigger).
 * You can call this from your admin panel (POST) — keep it protected behind auth in production.
 */
app.post('/sync-qikink', async (req, res) => {
  try {
    // Get token
    const token = await getQikToken();
    // Fetch products
    const items = await fetchQikProducts(token);

    // Map & upsert each item
    let success = 0, failed = 0;
    for (const item of items) {
      try {
        const doc = mapProductToDoc(item);
        await upsertProductToFirestore(doc);
        success++;
      } catch (e) {
        console.error('Product upsert failed for item:', item, e.message);
        failed++;
      }
    }

    res.json({ ok: true, message: 'Sync complete', total: items.length, success, failed });
  } catch (err) {
    res.status(500).json({ ok: false, error: String(err.message || err) });
  }
});

/** health */
app.get('/', (req, res) => res.send('Qikink → Firestore sync server running'));

app.listen(PORT, () => {
  console.log(`Server listening on ${PORT}`);
});
// Firebase setup import { initializeApp } from "firebase/app"; import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword } from "firebase/auth"; import { getFirestore, doc, setDoc, getDoc, collection, addDoc, getDocs, updateDoc, deleteDoc } from "firebase/firestore";

const firebaseConfig = { apiKey: "AIzaSyD8DllQ3VICJMZSvN-rPTjl4DuPx30JYlo", authDomain: "elitetips-a0a0d.firebaseapp.com", projectId: "elitetips-a0a0d", storageBucket: "elitetips-a0a0d.firebasestorage.app", messagingSenderId: "799283183526", appId: "1:799283183526:web:8010bc7f1f0ce1bccd1967", measurementId: "G-LQQJ94WWX2" };

const app = initializeApp(firebaseConfig); const auth = getAuth(app); const db = getFirestore(app);

// React app starts import { useState, useEffect } from "react"; import { Button } from "@/components/ui/button"; import { Card, CardContent } from "@/components/ui/card"; import { Switch } from "@/components/ui/switch"; import { Moon, Sun, Trash2, Pencil } from "lucide-react"; import { Input } from "@/components/ui/input"; import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

export default function EliteTipsHome() { const [darkMode, setDarkMode] = useState(false); const [loggedIn, setLoggedIn] = useState(false); const [email, setEmail] = useState(""); const [password, setPassword] = useState(""); const [amount, setAmount] = useState(20); const [hasPaid, setHasPaid] = useState(false); const [predictions, setPredictions] = useState([]); const [newPrediction, setNewPrediction] = useState({ match: "", tip: "", confidence: "" }); const [editingId, setEditingId] = useState(null); const [payments, setPayments] = useState([]);

const isAdmin = email === "brysonmah1@gmail.com";

useEffect(() => { const script = document.createElement("script"); script.src = "https://js.paystack.co/v1/inline.js"; script.async = true; document.body.appendChild(script); }, []);

const fetchPredictions = async () => { const snap = await getDocs(collection(db, "predictions")); const data = snap.docs.map(doc => ({ id: doc.id, ...doc.data() })); setPredictions(data); };

const fetchPayments = async () => { const snap = await getDocs(collection(db, "payments")); const data = snap.docs.map(doc => ({ email: doc.id, ...doc.data() })); setPayments(data); };

const checkPaymentStatus = async (userEmail) => { const ref = doc(db, "payments", userEmail); const snap = await getDoc(ref); if (snap.exists()) setHasPaid(true); else setHasPaid(false); fetchPredictions(); };

const handleLogin = async () => { try { const res = await signInWithEmailAndPassword(auth, email, password); setLoggedIn(true); checkPaymentStatus(res.user.email); fetchPayments(); } catch { alert("Login failed"); } };

const handleSignup = async () => { try { const res = await createUserWithEmailAndPassword(auth, email, password); setLoggedIn(true); } catch { alert("Signup failed"); } };

const handlePaystack = () => { const handler = window.PaystackPop.setup({ key: "pk_test_113b88ba8174bcfc85f8b9795dbf0157aa35a60d", email: email, amount: amount * 100, currency: "KES", callback: async function (response) { await setDoc(doc(db, "payments", email), { reference: response.reference, amount, timestamp: new Date().toISOString() }); alert("Payment successful!"); setHasPaid(true); fetchPredictions(); fetchPayments(); }, onClose: function () { alert("Payment closed"); }, }); handler.openIframe(); };

const handleAddOrUpdatePrediction = async () => { if (!newPrediction.match || !newPrediction.tip) return; if (editingId) { await updateDoc(doc(db, "predictions", editingId), newPrediction); setEditingId(null); } else { await addDoc(collection(db, "predictions"), newPrediction); } setNewPrediction({ match: "", tip: "", confidence: "" }); fetchPredictions(); };

const handleEdit = (prediction) => { setNewPrediction({ match: prediction.match, tip: prediction.tip, confidence: prediction.confidence }); setEditingId(prediction.id); };

const handleDelete = async (id) => { await deleteDoc(doc(db, "predictions", id)); fetchPredictions(); };

return ( <div className={darkMode ? "bg-black text-white min-h-screen" : "bg-white text-black min-h-screen"}> <header className="flex justify-between items-center p-4 shadow-md"> <h1 className="text-2xl font-bold">EliteTips</h1> <div className="flex items-center gap-2"> <span>{darkMode ? <Moon /> : <Sun />}</span> <Switch checked={darkMode} onCheckedChange={() => setDarkMode(!darkMode)} /> </div> </header>

<main className="p-4 grid gap-6">
    <Tabs defaultValue="predictions" className="w-full">
      <TabsList className="grid grid-cols-4 gap-2">
        <TabsTrigger value="predictions">Predictions</TabsTrigger>
        <TabsTrigger value="subscribe">Subscribe</TabsTrigger>
        <TabsTrigger value="login">Login</TabsTrigger>
        <TabsTrigger value="admin">Admin</TabsTrigger>
      </TabsList>

      <TabsContent value="predictions">
        <Card className="rounded-2xl shadow-xl p-4">
          <CardContent>
            <h2 className="text-xl font-semibold mb-2">Today's Predictions</h2>
            {hasPaid ? (
              <ul className="list-disc pl-4 space-y-1">
                {predictions.map((pred, i) => (
                  <li key={i}>
                    <strong>{pred.match}</strong>: {pred.tip} {pred.confidence && `(${pred.confidence})`}
                  </li>
                ))}
              </ul>
            ) : (
              <>
                <p>Subscribe to unlock today's expert predictions.</p>
                {!loggedIn && <Button className="mt-4">Login First</Button>}
              </>
            )}
          </CardContent>
        </Card>
      </TabsContent>

      <TabsContent value="subscribe">
        <Card className="rounded-2xl shadow-xl p-4">
          <CardContent>
            <h2 className="text-xl font-semibold mb-2">Subscription Plans</h2>
            <ul className="space-y-2 mb-4">
              <li><strong>KES 20</strong> – Single tip access</li>
              <li><strong>KES 50</strong> – Daily tips</li>
              <li><strong>KES 75</strong> – 3-day premium access</li>
              <li><strong>KES 100</strong> – Weekly expert picks</li>
              <li><strong>KES 150</strong> – Full premium month</li>
            </ul>
            <select
              className="p-2 border rounded mb-4 w-full text-black"
              value={amount}
              onChange={(e) => setAmount(Number(e.target.value))}
            >
              <option value={20}>KES 20</option>
              <option value={50}>KES 50</option>
              <option value={75}>KES 75</option>
              <option value={100}>KES 100</option>
              <option value={150}>KES 150</option>
            </select>
            <Button onClick={handlePaystack}>
              Pay KES {amount} with Paystack
            </Button>
          </CardContent>
        </Card>
      </TabsContent>

      <TabsContent value="login">
        <Card className="rounded-2xl shadow-xl p-4 max-w-md mx-auto">
          <CardContent>
            <h2 className="text-xl font-semibold mb-4">Login or Sign Up</h2>
            <Input type="email" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} className="mb-2" />
            <Input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} className="mb-4" />
            <div className="flex gap-2">
              <Button onClick={handleLogin}>Login</Button>
              <Button onClick={handleSignup} variant="outline">Sign Up</Button>
            </div>
          </CardContent>
        </Card>
      </TabsContent>

      <TabsContent value="admin">
        <Card className="rounded-2xl shadow-xl p-4 max-w-3xl mx-auto">
          <CardContent>
            <h2 className="text-xl font-semibold mb-4">Admin Dashboard</h2>
            {isAdmin ? (
              <>
                <Input
                  placeholder="Match Title"
                  value={newPrediction.match}
                  onChange={(e) => setNewPrediction({ ...newPrediction, match: e.target.value })}
                  className="mb-2"
                />
                <Input
                  placeholder="Prediction Tip"
                  value={newPrediction.tip}
                  onChange={(e) => setNewPrediction({ ...newPrediction, tip: e.target.value })}
                  className="mb-2"
                />
                <Input
                  placeholder="Confidence (optional)"
                  value={newPrediction.confidence}
                  onChange={(e) => setNewPrediction({ ...newPrediction, confidence: e.target.value })}
                  className="mb-4"
                />
                <Button onClick={handleAddOrUpdatePrediction}>
                  {editingId ? "Update Prediction" : "Add Prediction"}
                </Button>

                <h3 className="text-lg mt-6 mb-2">Manage Predictions</h3>
                <ul className="space-y-2">
                  {predictions.map((pred) => (
                    <li key={pred.id} className="flex justify-between border p-2 rounded">
                      <div>
                        <strong>{pred.match}</strong>: {pred.tip} {pred.confidence && `(${pred.confidence})`}
                      </div>
                      <div className="flex gap-2">
                        <Button size="sm" onClick={() => handleEdit(pred)}><Pencil size={16} /></Button>
                        <Button size="sm" variant="destructive" onClick={() => handleDelete(pred.id)}><Trash2 size={16} /></Button>
                      </div>
                    </li>
                  ))}
                </ul>

                <h3 className="text-lg mt-6 mb-2">Payment History</h3>
                <ul className="space-y-1">
                  {payments.map((p, i) => (
                    <li key={i}>
                      {p.email}: KES {p.amount} – {new Date(p.timestamp).toLocaleString()}
                    </li>
                  ))}
                </ul>
              </>
            ) : (
              <p>You are not authorized to access this page.</p>
            )}
          </CardContent>
        </Card>
      </TabsContent>
    </Tabs>
  </main>
</div>

); }

# Elitetips2025v

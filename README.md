// pages/dashboard.js import { useEffect, useState } from "react"; import { auth, db } from "../lib/firebase"; import { collection, addDoc, query, where, getDocs, doc, deleteDoc, updateDoc } from "firebase/firestore"; import { onAuthStateChanged, signOut } from "firebase/auth"; import { useRouter } from "next/router"; import jsPDF from "jspdf"; export default function Dashboard() { const [user, setUser] = useState(null); const [judul, setJudul] = useState(""); const [lirik, setLirik] = useState(""); const [laguList, setLaguList] = useState([]); const [editingId, setEditingId] = useState(null); const router = useRouter(); useEffect(() => { const unsub = onAuthStateChanged(auth, (user) => { if (user) { setUser(user); fetchLagu(user.uid); } else { router.push("/login"); } }); return () => unsub(); }, []); const fetchLagu = async (uid) => { const q = query(collection(db, "lagu"), where("uid", "==", uid)); const snapshot = await getDocs(q); const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })); setLaguList(data); }; const handleAddOrUpdate = async () => { if (!judul.trim() || !lirik.trim()) return; if (editingId) { await updateDoc(doc(db, "lagu", editingId), { judul, lirik }); setEditingId(null); } else { await addDoc(collection(db, "lagu"), { uid: user.uid, judul, lirik }); } setJudul(""); setLirik(""); fetchLagu(user.uid); }; const handleDelete = async (id) => { await deleteDoc(doc(db, "lagu", id)); fetchLagu(user.uid); }; const handleEdit = (lagu) => { setJudul(lagu.judul); setLirik(lagu.lirik); setEditingId(lagu.id); }; const handleLogout = () => { signOut(auth); router.push("/login"); }; const exportPDF = (lagu) => { const doc = new jsPDF(); doc.text(lagu.judul, 10, 10); doc.text(lagu.lirik, 10, 20); doc.save(`${lagu.judul}.pdf`); }; return ( 

Dashboard Lagu Gereja Logout 

setJudul(e.target.value)} /> setLirik(e.target.value)} /> {editingId ? "Update" : "Tambah"} 

{laguList.map(lagu => ( 

{lagu.judul} 

{lagu.lirik}

handleEdit(lagu)} className="text-blue-500">Edit handleDelete(lagu.id)} className="text-red-500">Hapus exportPDF(lagu)} className="text-green-600">PDF 

))} 

); } 

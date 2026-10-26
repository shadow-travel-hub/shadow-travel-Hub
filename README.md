<form id="reviewForm">
  <input id="rName" placeholder="Your name" maxlength="40" required>
  <select id="rRating">
    <option value="5">★★★★★ Excellent</option>
    <option value="4">★★★★ Very good</option>
    <option value="3">★★★ Good</option>
    <option value="2">★★ Fair</option>
    <option value="1">★ Poor</option>
  </select>
  <textarea id="rText" placeholder="Your review" maxlength="400" required></textarea>
  <button type="submit">Submit Review</button>
  <p id="rMsg"></p>
</form>
<div id="reviewList"></div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
import { getFirestore, collection, addDoc, query, where, onSnapshot, serverTimestamp }
  from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

const app = initializeApp({ /* paste your Firebase config here */ });
const db = getFirestore(app);
const col = collection(db, "reviews");

document.getElementById("reviewForm").addEventListener("submit", async (e) => {
  e.preventDefault();
  await addDoc(col, {
    name: rName.value.trim(),
    rating: Number(rRating.value),
    text: rText.value.trim(),
    approved: false,
    createdAt: serverTimestamp()
  });
  e.target.reset();
  rMsg.textContent = "Thank you! Your review will appear after approval.";
});

onSnapshot(query(col, where("approved", "==", true)), (snap) => {
  const list = document.getElementById("reviewList");
  list.innerHTML = "";
  snap.docs
    .map(d => d.data())
    .sort((a, b) => (b.createdAt?.seconds || 0) - (a.createdAt?.seconds || 0))
    .forEach(r => {
      const card = document.createElement("div");
      card.className = "review-card";
      const stars = document.createElement("div");
      stars.textContent = "★".repeat(r.rating) + "☆".repeat(5 - r.rating);
      const p = document.createElement("p");
      p.textContent = r.text;
      const n = document.createElement("strong");
      n.textContent = "- " + r.name;
      card.append(stars, p, n);
      list.appendChild(card);
    });
});
</script>

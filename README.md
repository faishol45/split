<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BagiBayar Ultra | All-in-One</title>
    <style>
        /* CSS SECTION */
        :root {
            --primary: #4f46e5;
            --primary-light: #eef2ff;
            --danger: #ef4444;
            --success: #10b981;
            --dark: #1e293b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f3f4f6;
            color: #374151;
            display: flex;
            justify-content: center;
            padding: 20px;
            margin: 0;
        }

        .container {
            width: 100%;
            max-width: 600px;
        }

        h1 {
            text-align: center;
            color: var(--primary);
            margin-bottom: 30px;
        }

        .card {
            background: white;
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.05);
            margin-bottom: 20px;
        }

        h3 {
            margin-top: 0;
            font-size: 1.1rem;
            border-bottom: 2px solid var(--primary-light);
            padding-bottom: 10px;
            margin-bottom: 15px;
        }

        .input-row {
            display: flex;
            gap: 10px;
        }

        input {
            flex: 1;
            padding: 12px;
            border: 1px solid #d1d5db;
            border-radius: 8px;
            font-size: 1rem;
            outline: none;
        }

        input:focus {
            border-color: var(--primary);
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
        }

        button {
            padding: 10px 20px;
            border-radius: 8px;
            border: none;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
        }

        .btn-add { background: var(--primary); color: white; }
        .btn-item { background: var(--success); color: white; }
        .btn-del { background: none; color: var(--danger); font-size: 0.85rem; padding: 5px; }

        .name-chips {
            display: flex;
            gap: 8px;
            flex-wrap: wrap;
            margin-top: 10px;
        }

        .chip {
            padding: 6px 14px;
            border: 2px solid var(--primary);
            border-radius: 20px;
            cursor: pointer;
            font-size: 0.9rem;
            font-weight: 500;
            display: flex;
            align-items: center;
            gap: 8px;
            transition: 0.2s;
        }

        .chip.active {
            background: var(--primary);
            color: white;
        }

        .chip-remove {
            font-weight: bold;
            font-size: 1.2rem;
            line-height: 1;
        }

        .item-card {
            border-left: 6px solid var(--success);
            transition: transform 0.2s;
        }

        .item-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }

        .result-card {
            background: var(--dark);
            color: white;
        }

        .total-row {
            display: flex;
            justify-content: space-between;
            padding: 12px 0;
            border-bottom: 1px solid #334155;
        }

        .total-row:last-child { border: none; }

        .empty-state {
            color: #9ca3af;
            text-align: center;
            font-style: italic;
            font-size: 0.9rem;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>BagiBayar Ultra</h1>
    
    <div class="card">
        <h3>1. Kelola Teman</h3>
        <div class="input-row">
            <input type="text" id="personName" placeholder="Ketik nama...">
            <button class="btn-add" onclick="addPerson()">Tambah</button>
        </div>
        <div id="peopleList" class="name-chips"></div>
    </div>

    <div class="card">
        <h3>2. Input Pesanan</h3>
        <div class="input-row" style="margin-bottom: 10px;">
            <input type="text" id="itemName" placeholder="Nama menu (misal: Pizza)">
            <input type="number" id="itemPrice" placeholder="Harga">
        </div>
        <button class="btn-item" style="width: 100%;" onclick="addItem()">Tambah ke Daftar Belanja</button>
    </div>

    <div id="itemsContainer"></div>

    <div id="finalResult" class="card result-card" style="display:none;">
        <h3>Ringkasan Patungan</h3>
        <div id="individualTotals"></div>
    </div>
</div>

<script>
    /* JAVASCRIPT SECTION */
    let people = [];
    let items = [];

    // --- Fungsi Orang ---
    function addPerson() {
        const input = document.getElementById('personName');
        const name = input.value.trim();
        if (name && !people.includes(name)) {
            people.push(name);
            input.value = '';
            renderAll();
        }
    }

    function removePerson(name) {
        people = people.filter(p => p !== name);
        // Hapus nama ini dari semua item yang sudah dipilih
        items.forEach(item => {
            item.sharedWith = item.sharedWith.filter(p => p !== name);
        });
        renderAll();
    }

    // --- Fungsi Barang ---
    function addItem() {
        const nameIn = document.getElementById('itemName');
        const priceIn = document.getElementById('itemPrice');
        
        const name = nameIn.value.trim();
        const price = parseFloat(priceIn.value);

        if (!name || isNaN(price) || price <= 0) {
            alert("Masukkan nama dan harga yang valid!");
            return;
        }

        items.push({
            id: Date.now(),
            name: name,
            price: price,
            sharedWith: []
        });

        nameIn.value = '';
        priceIn.value = '';
        renderAll();
    }

    function deleteItem(id) {
        items = items.filter(i => i.id !== id);
        renderAll();
    }

    function toggleSelect(itemId, personName) {
        const item = items.find(i => i.id === itemId);
        const idx = item.sharedWith.indexOf(personName);
        
        if (idx > -1) {
            item.sharedWith.splice(idx, 1);
        } else {
            item.sharedWith.push(personName);
        }
        renderAll();
    }

    // --- Fungsi Render & Hitung ---
    function renderAll() {
        renderPeople();
        renderItems();
        calculateFinal();
    }

    function renderPeople() {
        const container = document.getElementById('peopleList');
        if (people.length === 0) {
            container.innerHTML = '<span class="empty-state">Belum ada orang yang ditambah.</span>';
            return;
        }
        container.innerHTML = people.map(p => `
            <div class="chip active">
                ${p} 
                <span class="chip-remove" onclick="removePerson('${p}')">&times;</span>
            </div>
        `).join('');
    }

    function renderItems() {
        const container = document.getElementById('itemsContainer');
        if (items.length === 0) {
            container.innerHTML = '';
            return;
        }

        container.innerHTML = items.map(item => `
            <div class="card item-card">
                <div class="item-header">
                    <div>
                        <strong>${item.name}</strong><br>
                        <small>Rp ${item.price.toLocaleString('id-ID')}</small>
                    </div>
                    <button class="btn-del" onclick="deleteItem(${item.id})">Hapus</button>
                </div>
                <div class="name-chips">
                    ${people.length === 0 ? '<small class="empty-state">Tambahkan orang di atas dulu untuk memilih.</small>' : 
                      people.map(p => `
                        <div class="chip ${item.sharedWith.includes(p) ? 'active' : ''}" 
                             onclick="toggleSelect(${item.id}, '${p}')">${p}</div>
                    `).join('')}
                </div>
            </div>
        `).join('');
    }

    function calculateFinal() {
        const resultCard = document.getElementById('finalResult');
        const resultDiv = document.getElementById('individualTotals');
        
        const totals = {};
        people.forEach(p => totals[p] = 0);

        items.forEach(item => {
            if (item.sharedWith.length > 0) {
                const splitPrice = item.price / item.sharedWith.length;
                item.sharedWith.forEach(p => totals[p] += splitPrice);
            }
        });

        resultDiv.innerHTML = '';
        let hasData = false;
        
        for (const [name, amount] of Object.entries(totals)) {
            if (amount > 0) {
                hasData = true;
                resultDiv.innerHTML += `
                    <div class="total-row">
                        <span>${name}</span>
                        <strong>Rp ${Math.round(amount).toLocaleString('id-ID')}</strong>
                    </div>
                `;
            }
        }

        resultCard.style.display = hasData ? 'block' : 'none';
    }
</script>

</body>
</html>

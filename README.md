<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Minha Banca Oficial - Copa 2026</title>
    <style>
        :root { --primary: #8a1538; --success: #28a745; --bg: #f4f4f9; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); margin: 0; padding: 20px; padding-bottom: 100px; }
        header { background: var(--primary); color: white; padding: 20px; border-radius: 15px; text-align: center; margin-bottom: 25px; }
        
        /* Painel de Controle de Nuvem Simulado */
        .sync-bar { background: #333; color: #fff; padding: 10px; border-radius: 8px; margin-bottom: 15px; font-size: 12px; display: flex; justify-content: space-between; align-items: center; }
        
        .filtros { background: white; padding: 20px; border-radius: 12px; margin-bottom: 25px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        select { width: 100%; padding: 12px; border-radius: 8px; border: 1px solid #ccc; font-size: 16px; font-weight: bold; }
        .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 15px; }
        .card { background: white; border-radius: 15px; padding: 15px; text-align: center; border: 1px solid #ddd; }
        .card.esgotado { opacity: 0.4; filter: grayscale(1); }
        .card img { width: 100%; border-radius: 8px; margin-bottom: 10px; background: #eee; min-height: 180px; }
        
        .controles-dono { background: #f0f0f0; padding: 10px; border-radius: 8px; margin-top: 10px; border: 1px solid #ccc; }
        .input-preco { width: 50px; padding: 5px; border-radius: 4px; border: 1px solid #bbb; }
        
        .btn { width: 100%; padding: 10px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; margin-top: 5px; }
        .btn-add { background: var(--success); color: white; }
        .btn-status { background: #555; color: white; font-size: 10px; }
        .carrinho-fixo { position: fixed; bottom: 20px; right: 20px; background: var(--success); color: white; border: none; padding: 15px 25px; border-radius: 50px; cursor: pointer; font-weight: bold; z-index: 1000; }
    </style>
</head>
<body>

<div class="sync-bar">
    <span>📡 Status: Dados salvos localmente</span>
    <button onclick="exportarDados()" style="cursor:pointer; background:#555; color:white; border:none; padding:5px 10px; border-radius:4px;">Gerar Link de Backup</button>
</div>

<header>
    <h1>🏆 Catálogo Copa 2026 - Administrável</h1>
</header>

<div class="filtros">
    <select id="escolhaPais" onchange="renderizar()">
        <optgroup label="Seleções Principais">
            <option value="BRA">Brasil</option><option value="ARG">Argentina</option>
            <option value="FRA">França</option><option value="POR">Portugal</option>
            <option value="USA">EUA</option><option value="CAN">Canadá</option>
            <option value="MEX">México</option>
        </optgroup>
        <optgroup label="Especiais">
            <option value="COKE">🥤 Coca-Cola</option>
        </optgroup>
    </select>
</div>

<div class="grid" id="containerFigurinhas"></div>

<button class="carrinho-fixo" onclick="finalizarPedido()">
    🛒 Finalizar Pedido (<span id="totalItens">0</span>)
</button>

<script>
    // SISTEMA DE MEMÓRIA PERSISTENTE
    let bancoLoja = JSON.parse(localStorage.getItem('loja_2026_cloud')) || {};
    let meuCarrinho = [];

    function renderizar() {
        const grid = document.getElementById('containerFigurinhas');
        const pais = document.getElementById('escolhaPais').value;
        grid.innerHTML = '';

        let limite = (pais === 'COKE') ? 8 : 20;
        let prefix = (pais === 'COKE') ? 'C' : pais;

        for (let i = 1; i <= limite; i++) {
            const id = `${prefix}-${i}`;
            
            // Define valores iniciais se for a primeira vez
            if (!bancoLoja[id]) {
                let pPadrao = (pais === 'COKE') ? 5.00 : 1.00;
                bancoLoja[id] = { valor: pPadrao, disponivel: true };
            }

            const dados = bancoLoja[id];
            const card = document.createElement('div');
            card.className = `card ${dados.disponivel ? '' : 'esgotado'}`;

            card.innerHTML = `
                <img src="https://api.panini.com/stickers/2026/${pais}/${i}.jpg" onerror="this.src='https://via.placeholder.com/150x200/8a1538/ffffff?text=${id}'">
                <strong>${id}</strong>
                
                <div class="controles-dono">
                    R$ <input type="number" step="0.50" class="input-preco" value="${dados.valor}" 
                        onchange="salvarAlt('${id}', 'valor', this.value)">
                    <button class="btn btn-status" onclick="salvarAlt('${id}', 'disponivel', !${dados.disponivel})">
                        ${dados.disponivel ? '✅ Em Estoque' : '❌ Esgotado'}
                    </button>
                </div>

                <button class="btn btn-add" onclick="colocarNoCarrinho('${id}', ${dados.valor})" ${dados.disponivel ? '' : 'disabled'}>
                    ${dados.disponivel ? '🛒 Adicionar' : 'Indisponível'}
                </button>
            `;
            grid.appendChild(card);
        }
    }

    function salvarAlt(id, campo, valor) {
        if (campo === 'valor') bancoLoja[id].valor = parseFloat(valor);
        if (campo === 'disponivel') bancoLoja[id].disponivel = valor;
        
        localStorage.setItem('loja_2026_cloud', JSON.stringify(bancoLoja));
        renderizar();
    }

    function colocarNoCarrinho(id, preco) {
        meuCarrinho.push({id, preco});
        document.getElementById('totalItens').innerText = meuCarrinho.length;
    }

    function exportarDados() {
        const dadosStr = JSON.stringify(bancoLoja);
        prompt("Copie este código para salvar suas alterações permanentemente no arquivo HTML (cole na seção 'bancoLoja' do script):", dadosStr);
    }

    function finalizarPedido() {
        if (meuCarrinho.length === 0) return alert("Carrinho vazio!");
        let total = meuCarrinho.reduce((acc, curr) => acc + curr.preco, 0);
        let lista = meuCarrinho.map(i => i.id).join(", ");
        window.open(`https://wa.me/5541999999999?text=Gostaria de comprar: ${lista}. Total: R$ ${total.toFixed(2)}`);
    }

    renderizar();
</script>
</body>
</html>

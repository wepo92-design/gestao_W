<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Convite / Cobrança</title>
  <style>
    body { margin:0; padding:0; background:#E9F8F2; font-family: Arial, sans-serif; color:#1f2937; }
    .wrapper { display:flex; justify-content:center; align-items:flex-start; padding:24px; }
    .container { max-width:640px; width:100%; background:#fff; border-radius:16px; box-shadow:0 6px 24px rgba(0,0,0,.08); overflow:hidden; }
    .header { background:#00A86B; color:#fff; padding:24px; }
    .header h1 { margin:0; font-size:24px; }
    .content { padding:24px; }
    .highlight { background:#F4FBF8; border-left:4px solid #00A86B; padding:12px 14px; border-radius:10px; margin:16px 0; }
    .details { width:100%; border-collapse:collapse; margin:16px 0; }
    .details th { text-align:left; padding:8px 0; font-weight:bold; color:#111827; }
    .details td { padding:8px 0; color:#374151; }
    .cta { display:inline-block; padding:14px 20px; border-radius:12px; background:#00A86B; color:#fff; text-decoration:none; font-weight:700; }
    .cta:hover { background:#008a58; }
    .muted { color:#6b7280; font-size:13px; }
    .badge { display:inline-block; padding:6px 10px; border-radius:999px; background:#E9F8F2; color:#036947; font-weight:700; font-size:12px; }
    .price { font-size:26px; font-weight:800; color:#065f46; }
    .box { padding:16px; border:1px dashed #b7e7d3; border-radius:12px; background:#F7FFFB; margin:16px 0; }
    .info-extra { background:#F0FFF4; border-left:4px solid #00A86B; padding:10px; margin:10px 0; border-radius:10px; }
  </style>
</head>
<body>
  <div class="wrapper">
    <div class="container">
      <div class="header">
        <h1 id="titulo">Carregando...</h1>
      </div>
      <div class="content" id="conteudo">
        Aguarde, buscando informações...
      </div>
    </div>
  </div>

  <script>
    async function carregarDados() {
      const params = new URLSearchParams(window.location.search);
      const nome = params.get("nome") || "";
      const id   = params.get("id") || "";

      const apiUrl = "https://script.google.com/macros/s/AKfycbz6uPjAidaOA8gUoD3rR-bwgZ2aM34XfPDu8mfOV-VMuupnVi1Lm4KFf_7S156Z8LMx/exec?" +
        (id ? "id=" + encodeURIComponent(id) : "nome=" + encodeURIComponent(nome));

      try {
        const resp = await fetch(apiUrl);
        const dados = await resp.json();

        if (dados.erro) {
          document.getElementById("titulo").innerText = "Erro";
          document.getElementById("conteudo").innerHTML = "<p>" + dados.erro + "</p>";
          return;
        }

        if (dados.tipo === "cobranca") {
          renderizarCobranca(dados);
        } else {
          renderizarConvite(dados);
        }

      } catch (e) {
        document.getElementById("titulo").innerText = "Erro";
        document.getElementById("conteudo").innerHTML = "<p>Não foi possível carregar os dados.</p>";
      }
    }

    function renderizarConvite(d) {
      document.getElementById("titulo").innerText = "Convite Especial";
      let detalhes = `
        <tr><th>Endereço</th><td>${d.endereco || "-"}</td></tr>
        <tr><th>Traje</th><td>${d.traje || "Livre"}</td></tr>
      `;

      // adiciona automaticamente qualquer campo extra
      for (let key in d) {
        if (!["tipo","nome","evento","data","hora","local","endereco","traje","telefone"].includes(key)) {
          detalhes += `<tr><th>${key.charAt(0).toUpperCase() + key.slice(1)}</th><td>${d[key]}</td></tr>`;
        }
      }

      document.getElementById("conteudo").innerHTML = `
        <h2>Olá, ${d.nome}! 🎉</h2>
        <p class="muted">É com alegria que convidamos você para <b>${d.evento}</b>.</p>
        <div class="highlight">
          <strong>${d.evento}</strong><br/>
          ${d.data} às ${d.hora} • ${d.local}
        </div>
        <table class="details">
          ${detalhes}
        </table>
        <p><a class="cta" href="https://wa.me/${d.telefone}?text=${encodeURIComponent("Confirmo presença no evento " + d.evento)}" target="_blank">Confirmar pelo WhatsApp</a></p>
      `;
    }

    function renderizarCobranca(d) {
      document.getElementById("titulo").innerText = "Cobrança";

      let extras = "";
      for (let key in d) {
        if (!["tipo","nome","descricao","valor","vencimento","fatura","telefone","linkPagamento"].includes(key)) {
          extras += `<div class="info-extra"><strong>${key.charAt(0).toUpperCase() + key.slice(1)}:</strong> ${d[key]}</div>`;
        }
      }

      document.getElementById("conteudo").innerHTML = `
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <h2>Olá, ${d.nome}</h2>
          <span class="badge">Cobrança</span>
        </div>
        <p>Segue o detalhamento do pagamento referente a <strong>${d.descricao}</strong>.</p>
        <div class="box">
          <div class="price">${d.valor}</div>
          <div class="muted">Vencimento: <strong>${d.vencimento}</strong></div>
          <div class="muted">Identificador: <strong>${d.fatura}</strong></div>
          ${extras}
        </div>
        <p><a class="cta" href="${d.linkPagamento}" target="_blank">Pagar agora</a></p>
        <p class="muted">Dúvidas? <a href="https://wa.me/${d.telefone}?text=${encodeURIComponent("Tenho dúvidas sobre minha cobrança")}" target="_blank">Fale pelo WhatsApp</a></p>
      `;
    }

    carregarDados();
  </script>
</body>
</html>

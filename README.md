# geradordeorcamento.github.io-
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Gerador Pro de Propostas - Vinycst</title>
  
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- FontAwesome -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  
  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link id="dynamic-google-fonts" rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Anton&family=Bebas+Neue&family=Caveat:wght@600&family=Inter:wght@400;600;700&family=Kalam:wght@700&family=Montserrat:wght@800;900&family=Oswald:wght@700&family=Permanent+Marker&family=Plus+Jakarta+Sans:wght@400;600;700&family=Poppins:wght@400;600;700&family=Rock+Salt&family=Shadows+Into+Light&family=Space+Grotesk:wght@500;700&family=Syne:wght@800&family=Unbounded:wght@800&display=swap">

  <style>
    /* Variáveis Globais de Fonte */
    :root {
      --font-title-family: 'Bebas Neue';
      --font-highlight-family: 'Caveat';
      --font-body-family: 'Inter';
    }

    .font-title { font-family: var(--font-title-family), sans-serif; }
    .font-highlight { font-family: var(--font-highlight-family), cursive; }
    .font-body { font-family: var(--font-body-family), sans-serif; }

    .highlight-text {
      font-family: var(--font-highlight-family), cursive;
      color: var(--accent-color, #ec4899);
      font-size: 1.25em;
      display: inline-block;
      transform: rotate(-2deg);
    }

    /* Estilo de Edição Direta */
    [contenteditable="true"] {
      transition: background-color 0.2s;
      outline: none;
    }
    [contenteditable="true"]:hover {
      background-color: rgba(128,128,128,0.1);
      border-radius: 4px;
    }
    [contenteditable="true"]:focus {
      box-shadow: 0 0 0 2px rgba(99, 102, 241, 0.4);
      border-radius: 4px;
      background-color: rgba(128,128,128,0.1);
    }

    /* Tirar setas do input number no popup */
    input[type=number]::-webkit-inner-spin-button, 
    input[type=number]::-webkit-outer-spin-button { 
      -webkit-appearance: none; 
      margin: 0; 
    }

    /* FORÇAR CORES NA EXPORTAÇÃO PARA PDF */
    @media print {
      * {
        -webkit-print-color-adjust: exact !important;
        print-color-adjust: exact !important;
        color-adjust: exact !important;
      }
      @page { margin: 0; }
      body { margin: 0 !important; padding: 0 !important; background-color: transparent !important; }
      .no-print, #inline-editor { display: none !important; }
      .proposal-canvas { box-shadow: none !important; border: none !important; width: 100% !important; max-width: 100% !important; margin: 0 !important; min-height: 100vh; }
      .page-break { page-break-before: always; }
      section, .card-element, tr { break-inside: avoid; page-break-inside: avoid; }
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 font-body min-h-screen flex flex-col lg:flex-row relative">

  <!-- POP-UP FLUTUANTE (INLINE EDITOR) -->
  <div id="inline-editor" class="no-print hidden absolute z-50 bg-slate-800 border border-slate-600 rounded-lg shadow-2xl p-1.5 flex flex-wrap items-center gap-2 text-white transition-opacity">
    
    <!-- Controles de Tamanho -->
    <div class="flex items-center gap-1 bg-slate-900 rounded p-1">
      <button onclick="changeFontSize(-1)" class="w-6 h-6 hover:bg-slate-700 rounded flex items-center justify-center text-slate-300" title="Diminuir"><i class="fa-solid fa-minus text-[10px]"></i></button>
      <input type="number" id="inline-font-size" onchange="setFontSize(this.value)" class="w-8 bg-transparent text-center text-xs font-mono outline-none text-white">
      <button onclick="changeFontSize(1)" class="w-6 h-6 hover:bg-slate-700 rounded flex items-center justify-center text-slate-300" title="Aumentar"><i class="fa-solid fa-plus text-[10px]"></i></button>
    </div>

    <!-- Alinhamento -->
    <div class="flex items-center gap-1 bg-slate-900 rounded p-1">
      <button onclick="changeAlign('left')" class="w-6 h-6 hover:bg-slate-700 rounded flex items-center justify-center text-slate-300"><i class="fa-solid fa-align-left text-xs"></i></button>
      <button onclick="changeAlign('center')" class="w-6 h-6 hover:bg-slate-700 rounded flex items-center justify-center text-slate-300"><i class="fa-solid fa-align-center text-xs"></i></button>
      <button onclick="changeAlign('right')" class="w-6 h-6 hover:bg-slate-700 rounded flex items-center justify-center text-slate-300"><i class="fa-solid fa-align-right text-xs"></i></button>
    </div>

    <!-- Estilo da Fonte -->
    <div class="flex items-center gap-1 bg-slate-900 rounded p-1 text-[10px] font-bold uppercase">
      <button onclick="applyFontStyle('title')" class="px-2 h-6 hover:bg-slate-700 rounded text-slate-300">Título</button>
      <button onclick="applyFontStyle('highlight')" class="px-2 h-6 hover:bg-slate-700 rounded text-slate-300">Destaque</button>
      <button onclick="applyFontStyle('body')" class="px-2 h-6 hover:bg-slate-700 rounded text-slate-300">Corpo</button>
    </div>
  </div>


  <!-- Painel de Controles Lateral -->
  <aside class="no-print w-full lg:w-[420px] bg-slate-900 border-r border-slate-800 p-6 overflow-y-auto max-h-screen shadow-2xl flex-shrink-0">
    <div class="flex items-center justify-between mb-6 pb-4 border-b border-slate-800">
      <div class="flex items-center gap-3">
        <div class="p-2 bg-indigo-600 rounded-lg text-white">
          <i class="fa-solid fa-pen-nib text-xl"></i>
        </div>
        <div>
          <h1 class="font-bold text-lg text-white">Editor Pro</h1>
          <p class="text-xs text-slate-400">Clique nos textos para ajustar</p>
        </div>
      </div>
      <button onclick="window.print()" class="bg-indigo-600 hover:bg-indigo-500 text-white text-xs font-bold py-2 px-3 rounded-lg flex items-center gap-2 transition">
        <i class="fa-solid fa-file-pdf"></i> Gerar PDF
      </button>
    </div>

    <div class="space-y-6">

      <!-- Templates e Cores -->
      <div class="bg-slate-800/60 p-4 rounded-xl border border-slate-700/50">
        <h2 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 flex items-center gap-2">
          <i class="fa-solid fa-palette text-indigo-400"></i> Tema Visual
        </h2>
        <div class="grid grid-cols-3 gap-2 mb-4">
          <button onclick="applyPreset('neon')" class="p-2 text-xs bg-slate-900 hover:bg-slate-700 border border-slate-700 rounded-lg text-cyan-400 font-bold">Neon</button>
          <button onclick="applyPreset('dark')" class="p-2 text-xs bg-slate-900 hover:bg-slate-700 border border-slate-700 rounded-lg text-pink-500 font-bold">Dark</button>
          <button onclick="applyPreset('clean')" class="p-2 text-xs bg-slate-900 hover:bg-slate-700 border border-slate-700 rounded-lg text-emerald-400 font-bold">Clean</button>
        </div>
        <div class="grid grid-cols-2 gap-3 text-xs mt-4">
          <div><label class="block text-slate-400 mb-1">Fundo</label><input type="color" id="color-bg" value="#050508" oninput="updateGlobalColors()" class="w-full h-8 bg-slate-900 rounded cursor-pointer border border-slate-700"></div>
          <div><label class="block text-slate-400 mb-1">Cartões</label><input type="color" id="color-card" value="#111118" oninput="updateGlobalColors()" class="w-full h-8 bg-slate-900 rounded cursor-pointer border border-slate-700"></div>
          <div><label class="block text-slate-400 mb-1">Destaque</label><input type="color" id="color-accent" value="#ec4899" oninput="updateGlobalColors()" class="w-full h-8 bg-slate-900 rounded cursor-pointer border border-slate-700"></div>
          <div><label class="block text-slate-400 mb-1">Bordas</label><input type="color" id="color-border" value="#27272a" oninput="updateGlobalColors()" class="w-full h-8 bg-slate-900 rounded cursor-pointer border border-slate-700"></div>
        </div>
      </div>

      <!-- Seleção Global de Fontes -->
      <div class="bg-slate-800/60 p-4 rounded-xl border border-slate-700/50">
        <h2 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 flex items-center gap-2">
          <i class="fa-solid fa-font text-indigo-400"></i> Escolha as Fontes
        </h2>
        <div class="space-y-3">
          <div>
            <label class="block text-xs text-slate-400 mb-1">Fonte Principal (Títulos)</label>
            <select id="select-font-title" onchange="updateFontFamilies()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-xs text-white">
              <option value="Bebas Neue">Bebas Neue (Impactante)</option>
              <option value="Montserrat">Montserrat (Moderna)</option>
              <option value="Anton">Anton (Bold)</option>
            </select>
          </div>
          <div>
            <label class="block text-xs text-slate-400 mb-1">Fonte Manuscrita (Destaques)</label>
            <select id="select-font-highlight" onchange="updateFontFamilies()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-xs text-white">
              <option value="Caveat">Caveat (Urbana)</option>
              <option value="Permanent Marker">Permanent Marker</option>
              <option value="Shadows Into Light">Shadows Into Light</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Redes Sociais Dinâmicas -->
      <div class="bg-slate-800/60 p-4 rounded-xl border border-slate-700/50">
        <h2 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 flex items-center gap-2">
          <i class="fa-solid fa-hashtag text-indigo-400"></i> Redes Sociais
        </h2>
        <div class="space-y-2 text-xs">
          <div class="flex items-center gap-2"><i class="fa-brands fa-instagram text-slate-400 w-4"></i><input type="text" id="soc-ig" value="@vinycst" placeholder="@seuinstagram" oninput="updateSocials()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-white"></div>
          <div class="flex items-center gap-2"><i class="fa-brands fa-behance text-slate-400 w-4"></i><input type="text" id="soc-be" placeholder="behance.net/seuportfolio" oninput="updateSocials()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-white"></div>
          <div class="flex items-center gap-2"><i class="fa-brands fa-youtube text-slate-400 w-4"></i><input type="text" id="soc-yt" placeholder="youtube.com/canal" oninput="updateSocials()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-white"></div>
          <div class="flex items-center gap-2"><i class="fa-brands fa-facebook text-slate-400 w-4"></i><input type="text" id="soc-fb" placeholder="facebook.com/pagina" oninput="updateSocials()" class="w-full bg-slate-900 border border-slate-700 rounded p-1.5 text-white"></div>
        </div>
      </div>

      <!-- Ativar / Desativar Seções -->
      <div class="bg-slate-800/60 p-4 rounded-xl border border-slate-700/50">
        <h2 class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-3 flex items-center gap-2">
          <i class="fa-solid fa-toggle-on text-indigo-400"></i> Seções Opcionais
        </h2>
        <div class="space-y-2 text-xs text-slate-300">
          <label class="flex items-center gap-2 cursor-pointer"><input type="checkbox" id="toggle-portfolio" checked onchange="toggleSection('section-portfolio', this.checked)" class="rounded bg-slate-900 border-slate-700 text-indigo-500 focus:ring-indigo-500"> Exibir Portfólio</label>
          <label class="flex items-center gap-2 cursor-pointer"><input type="checkbox" id="toggle-comparative" onchange="toggleSection('section-comparative', this.checked)" class="rounded bg-slate-900 border-slate-700 text-indigo-500 focus:ring-indigo-500"> Exibir Tabela de Planos</label>
          <label class="flex items-center gap-2 cursor-pointer"><input type="checkbox" id="toggle-terms" checked onchange="toggleSection('section-terms', this.checked)" class="rounded bg-slate-900 border-slate-700 text-indigo-500 focus:ring-indigo-500"> Exibir Termos de Contrato</label>
        </div>
      </div>

    </div>
  </aside>

  <!-- ÁREA DE VISUALIZAÇÃO E EDIÇÃO DIRETA -->
  <main class="flex-grow p-0 lg:p-12 overflow-y-auto flex justify-center items-start" id="main-area">
    <div id="proposal-container" class="proposal-canvas w-full max-w-[850px] p-8 lg:p-12 rounded-2xl shadow-2xl transition-all relative border-2" style="background-color: #050508; border-color: #27272a;">

      <!-- CABEÇALHO -->
      <header class="mb-12 border-b pb-8 flex flex-col md:flex-row justify-between items-start md:items-end gap-6 border-opacity-50" style="border-color: inherit;">
        <div class="w-full">
          <span contenteditable="true" class="font-highlight mb-1 block px-1 text-left" style="color: var(--accent-color); font-size: 20px;">Projeto Exclusivo</span>
          <h1 contenteditable="true" class="font-title uppercase leading-none p-1 text-left" style="color: #ffffff; font-size: 58px;">PROPOSTA COMERCIAL</h1>
          <p contenteditable="true" class="font-body font-bold mt-2 p-1 text-left" style="color: var(--accent-color); font-size: 20px;">VIDEOMAKER & EDITOR</p>
        </div>
        <div class="text-right flex-shrink-0 flex flex-col items-end">
          <div id="social-container" class="flex gap-3 mb-2 text-sm"></div>
          <div contenteditable="true" class="font-body text-right p-1" style="color: #a1a1aa; font-size: 14px;">
            Recife, PE - 07 de Outubro de 2026<br>Contato: (81) 99144-8128
          </div>
        </div>
      </header>

      <!-- SEÇÃO: QUEM SOU EU -->
      <section class="mb-12 grid grid-cols-1 md:grid-cols-3 gap-8 items-center">
        <div class="md:col-span-2 space-y-4">
          <h2 contenteditable="true" class="font-title uppercase tracking-wide text-left p-1" style="color: #ffffff; font-size: 32px;">O Profissional</h2>
          <p contenteditable="true" class="font-body leading-relaxed p-1 text-left" style="color: #a1a1aa; font-size: 15px;">
            Sou Vinicius Costa, especialista em captação e edição de vídeo de alto impacto para grandes marcas e eventos. Trabalhamos com equipamentos de ponta, entregando qualidade 4K focada em narrativa visual, engajamento e color grading avançado.
          </p>
          <div class="pt-2 flex flex-wrap gap-2">
            <span class="px-3 py-1 rounded-lg text-xs font-bold border border-blue-500 text-blue-500 bg-blue-500/10">Pr</span>
            <span class="px-3 py-1 rounded-lg text-xs font-bold border border-purple-500 text-purple-500 bg-purple-500/10">Ae</span>
            <span class="px-3 py-1 rounded-lg text-xs font-bold border border-red-400 text-red-400 bg-red-400/10">DaVinci</span>
          </div>
        </div>
        <div class="flex flex-col items-center justify-center">
          <div class="relative group w-40 h-40 rounded-2xl overflow-hidden border-2 border-slate-700 shadow-xl bg-slate-900 flex items-center justify-center">
            <img id="profile-img" src="https://images.unsplash.com/photo-1534528741775-53994a69daeb?w=400&auto=format&fit=crop&q=80" class="w-full h-full object-cover">
            <label for="img-upload" class="no-print absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 flex flex-col items-center justify-center text-white text-xs cursor-pointer transition">
              <i class="fa-solid fa-camera mb-1"></i> Foto
            </label>
            <input type="file" id="img-upload" class="hidden" accept="image/*" onchange="loadProfileImage(event)">
          </div>
        </div>
      </section>

      <!-- PORTFÓLIO -->
      <section id="section-portfolio" class="mb-12 page-break">
        <h2 contenteditable="true" class="font-title uppercase tracking-wide mb-6 text-left p-1" style="color: #ffffff; font-size: 32px;">Trabalhos Recentes</h2>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div class="card-element p-3 rounded-xl border flex flex-col gap-2 relative" style="background-color: #111118; border-color: #27272a;">
            <div class="h-40 bg-black/30 rounded-lg flex items-center justify-center overflow-hidden relative group">
              <img id="port-img-1" src="" class="w-full h-full object-cover hidden absolute inset-0 z-0">
              <i class="fa-solid fa-play text-white text-3xl z-10 opacity-70 pointer-events-none drop-shadow-lg"></i>
              <label for="upload-port-1" class="no-print absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 flex flex-col items-center justify-center text-white text-xs cursor-pointer transition z-20"><i class="fa-solid fa-image mb-1"></i> Add Capa</label>
              <input type="file" id="upload-port-1" class="hidden" accept="image/*" onchange="loadPortfolioImage(event, 1)">
            </div>
            <h3 contenteditable="true" class="font-body font-bold text-left p-1" style="color: #ffffff; font-size: 18px;">Clipe Musical</h3>
            <div class="no-print w-full"><input type="text" placeholder="Cole o link do vídeo..." onchange="updatePortfolioLink(1, this.value)" class="w-full text-xs bg-slate-900 border border-slate-700 rounded p-1.5 text-white outline-none"></div>
            <a id="port-link-1" href="#" target="_blank" class="font-body text-xs hover:underline text-left font-bold mt-auto p-1" style="color: var(--accent-color);">Assistir Vídeo <i class="fa-solid fa-arrow-right text-[10px]"></i></a>
          </div>

          <div class="card-element p-3 rounded-xl border flex flex-col gap-2 relative" style="background-color: #111118; border-color: #27272a;">
            <div class="h-40 bg-black/30 rounded-lg flex items-center justify-center overflow-hidden relative group">
              <img id="port-img-2" src="" class="w-full h-full object-cover hidden absolute inset-0 z-0">
              <i class="fa-solid fa-play text-white text-3xl z-10 opacity-70 pointer-events-none drop-shadow-lg"></i>
              <label for="upload-port-2" class="no-print absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 flex flex-col items-center justify-center text-white text-xs cursor-pointer transition z-20"><i class="fa-solid fa-image mb-1"></i> Add Capa</label>
              <input type="file" id="upload-port-2" class="hidden" accept="image/*" onchange="loadPortfolioImage(event, 2)">
            </div>
            <h3 contenteditable="true" class="font-body font-bold text-left p-1" style="color: #ffffff; font-size: 18px;">Campanha Comercial</h3>
            <div class="no-print w-full"><input type="text" placeholder="Cole o link do vídeo..." onchange="updatePortfolioLink(2, this.value)" class="w-full text-xs bg-slate-900 border border-slate-700 rounded p-1.5 text-white outline-none"></div>
            <a id="port-link-2" href="#" target="_blank" class="font-body text-xs hover:underline text-left font-bold mt-auto p-1" style="color: var(--accent-color);">Assistir Vídeo <i class="fa-solid fa-arrow-right text-[10px]"></i></a>
          </div>
        </div>
      </section>

      <!-- TABELA COMPARATIVA -->
      <section id="section-comparative" class="mb-12 hidden page-break">
        <h2 contenteditable="true" class="font-title uppercase tracking-wide mb-6 text-left p-1" style="color: #ffffff; font-size: 32px;">Opções de Planos</h2>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div class="card-element p-6 rounded-xl border" style="background-color: #111118; border-color: #27272a;">
            <h3 contenteditable="true" class="font-body font-bold mb-4 text-left p-1" style="color: #ffffff; font-size: 22px;">Pacote Básico</h3>
            <ul class="space-y-2">
              <li contenteditable="true" class="font-body text-left p-1" style="color: #a1a1aa; font-size: 15px;"><i class="fa-solid fa-check text-emerald-500 mr-2"></i> 1 Diária de Captação</li>
              <li contenteditable="true" class="font-body text-left p-1" style="color: #a1a1aa; font-size: 15px;"><i class="fa-solid fa-xmark text-red-500 mr-2"></i> Sem Color Grading</li>
            </ul>
          </div>
          <div class="card-element p-6 rounded-xl border relative" style="background-color: #111118; border-color: #27272a;">
            <span class="absolute -top-3 left-4 text-white text-[10px] font-bold px-2 py-1 rounded uppercase tracking-wider" style="background-color: var(--accent-color);">Recomendado</span>
            <h3 contenteditable="true" class="font-body font-bold mb-4 text-left p-1" style="color: #ffffff; font-size: 22px;">Pacote Pro</h3>
            <ul class="space-y-2">
              <li contenteditable="true" class="font-body text-left p-1" style="color: #a1a1aa; font-size: 15px;"><i class="fa-solid fa-check text-emerald-500 mr-2"></i> Captação 4K Câmera + Drone</li>
              <li contenteditable="true" class="font-body text-left p-1" style="color: #a1a1aa; font-size: 15px;"><i class="fa-solid fa-check text-emerald-500 mr-2"></i> Color Grading Profissional</li>
            </ul>
          </div>
        </div>
      </section>

      <!-- TABELA DE ORÇAMENTO -->
      <section class="mb-12 page-break">
        <h2 contenteditable="true" class="font-title uppercase tracking-wide mb-4 text-left p-1" style="color: #ffffff; font-size: 32px;">Investimento</h2>

        <div class="overflow-x-auto card-element rounded-xl border" style="border-color: #27272a;">
          <table class="w-full text-left border-collapse">
            <thead style="background-color: rgba(128,128,128,0.1);">
              <tr class="border-b" style="border-color: inherit;">
                <th class="py-3 px-4"><div contenteditable="true" class="font-body font-bold uppercase tracking-wider p-1" style="color: #a1a1aa; font-size: 14px;">Descrição do Serviço</div></th>
                <th class="py-3 px-2 text-center w-20"><div contenteditable="true" class="font-body font-bold uppercase tracking-wider p-1 text-center" style="color: #a1a1aa; font-size: 14px;">Qtd</div></th>
                <th class="py-3 px-2 text-right w-32"><div contenteditable="true" class="font-body font-bold uppercase tracking-wider p-1 text-right" style="color: #a1a1aa; font-size: 14px;">Total</div></th>
              </tr>
            </thead>
            <tbody class="divide-y" style="border-color: inherit;">
              <tr class="hover:bg-black/10">
                <td class="py-3 px-4"><div contenteditable="true" class="font-body p-1" style="color: #a1a1aa; font-size: 15px;">Captação de Evento (Diária Completa)</div></td>
                <td class="py-3 px-2 text-center"><div contenteditable="true" class="font-body p-1 text-center" style="color: #a1a1aa; font-size: 15px;">1</div></td>
                <td class="py-3 px-2 text-right"><div contenteditable="true" class="font-body font-mono font-bold p-1 text-right" style="color: #ffffff; font-size: 15px;">1.200,00</div></td>
              </tr>
              <tr class="hover:bg-black/10">
                <td class="py-3 px-4"><div contenteditable="true" class="font-body p-1" style="color: #a1a1aa; font-size: 15px;">Edição Cinematográfica + 4K</div></td>
                <td class="py-3 px-2 text-center"><div contenteditable="true" class="font-body p-1 text-center" style="color: #a1a1aa; font-size: 15px;">1</div></td>
                <td class="py-3 px-2 text-right"><div contenteditable="true" class="font-body font-mono font-bold p-1 text-right" style="color: #ffffff; font-size: 15px;">800,00</div></td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mt-4 flex justify-end">
          <div class="w-full md:w-64 p-4 rounded-xl border card-element flex justify-between items-center" style="background-color: #111118; border-color: #27272a;">
            <span contenteditable="true" class="font-title p-1" style="color: #ffffff; font-size: 28px;">Total:</span>
            <span contenteditable="true" class="font-body font-mono font-bold p-1 text-right" style="color: var(--accent-color); font-size: 24px;">R$ 2.000,00</span>
          </div>
        </div>
      </section>

      <!-- TERMOS DE CONTRATO -->
      <section id="section-terms" class="mb-12 page-break">
        <h2 contenteditable="true" class="font-title uppercase tracking-wide mb-4 text-left p-1" style="color: #ffffff; font-size: 32px;">Termos & Condições</h2>
        <div class="card-element p-5 rounded-xl border space-y-3" style="background-color: #111118; border-color: #27272a;">
          <p contenteditable="true" class="font-body p-1 text-left" style="color: #a1a1aa; font-size: 14px;"><strong>1. Refações:</strong> Estão inclusas até 2 (duas) rodadas de alterações.</p>
          <p contenteditable="true" class="font-body p-1 text-left" style="color: #a1a1aa; font-size: 14px;"><strong>2. Prazos:</strong> Material entregue em até 10 dias úteis após a captação.</p>
          <p contenteditable="true" class="font-body p-1 text-left" style="color: #a1a1aa; font-size: 14px;"><strong>3. Cancelamento:</strong> Cancelamentos com menos de 48h não reembolsam o sinal.</p>
        </div>
      </section>

      <!-- RODAPÉ -->
      <footer class="pt-6 border-t flex flex-col md:flex-row justify-between gap-6" style="border-color: #27272a;">
        <div class="flex-1">
          <p contenteditable="true" class="font-body font-bold mb-1 text-left p-1" style="color: #ffffff; font-size: 16px;"><i class="fa-regular fa-calendar mr-2" style="color: var(--accent-color);"></i>Validade</p>
          <div contenteditable="true" class="font-body p-1 text-left" style="color: #a1a1aa; font-size: 14px;">15 dias a contar da emissão.</div>
        </div>
        <div class="flex-1">
          <p contenteditable="true" class="font-body font-bold mb-1 text-left p-1" style="color: #ffffff; font-size: 16px;"><i class="fa-solid fa-money-check-dollar mr-2" style="color: var(--accent-color);"></i>Pagamento</p>
          <div contenteditable="true" class="font-body p-1 text-left" style="color: #a1a1aa; font-size: 14px;">50% de sinal (PIX).<br>50% na entrega final.</div>
        </div>
      </footer>

    </div>
  </main>

  <script>
    // --- LÓGICA DO EDITOR INLINE (POP-UP) ---
    let activeEditable = null;
    const inlineEditor = document.getElementById('inline-editor');
    const fontSizeInput = document.getElementById('inline-font-size');

    // Escutar cliques em toda a tela
    document.addEventListener('click', (e) => {
      // Se clicou em um texto editável
      if (e.target.hasAttribute('contenteditable')) {
        activeEditable = e.target;
        showInlineEditor(activeEditable);
      } 
      // Se clicou fora do popup e fora de um editável, esconde o popup
      else if (!inlineEditor.contains(e.target)) {
        inlineEditor.classList.add('hidden');
        activeEditable = null;
      }
    });

    // Ao digitar no texto, reajusta a posição do popup
    document.addEventListener('input', (e) => {
      if (activeEditable && e.target === activeEditable) {
        showInlineEditor(activeEditable);
      }
    });

    function showInlineEditor(el) {
      const rect = el.getBoundingClientRect();
      // Posição vertical: acima do elemento. Se não couber, coloca embaixo.
      let top = window.scrollY + rect.top - 45; 
      if (rect.top < 50) top = window.scrollY + rect.bottom + 10;
      
      let left = window.scrollX + rect.left;

      inlineEditor.style.top = `${top}px`;
      inlineEditor.style.left = `${left}px`;
      inlineEditor.classList.remove('hidden');

      // Pega o tamanho atual da fonte do elemento clicado
      const style = window.getComputedStyle(el);
      fontSizeInput.value = parseInt(style.fontSize);
    }

    function changeFontSize(delta) {
      if (!activeEditable) return;
      const currentSize = parseInt(window.getComputedStyle(activeEditable).fontSize);
      const newSize = currentSize + delta;
      activeEditable.style.fontSize = `${newSize}px`;
      fontSizeInput.value = newSize;
      showInlineEditor(activeEditable); // Atualiza posição
    }

    function setFontSize(val) {
      if (!activeEditable || !val) return;
      activeEditable.style.fontSize = `${val}px`;
      showInlineEditor(activeEditable);
    }

    function changeAlign(align) {
      if (!activeEditable) return;
      activeEditable.style.textAlign = align;
    }

    function applyFontStyle(type) {
      if (!activeEditable) return;
      // Limpa as 3 classes
      activeEditable.classList.remove('font-title', 'font-highlight', 'font-body');
      
      // Aplica a nova classe selecionada
      if (type === 'title') activeEditable.classList.add('font-title');
      if (type === 'highlight') activeEditable.classList.add('font-highlight');
      if (type === 'body') activeEditable.classList.add('font-body');
    }


    // --- LÓGICA DE CORES E TEMA GLOBAL ---
    function updateGlobalColors() {
      const bg = document.getElementById('color-bg').value;
      const card = document.getElementById('color-card').value;
      const accent = document.getElementById('color-accent').value;
      const border = document.getElementById('color-border').value;

      document.documentElement.style.setProperty('--accent-color', accent);
      
      const container = document.getElementById('proposal-container');
      container.style.backgroundColor = bg;
      container.style.borderColor = border;

      document.querySelectorAll('.card-element').forEach(el => {
        el.style.backgroundColor = card;
        el.style.borderColor = border;
      });

      updateSocials();
    }

    function updateFontFamilies() {
      const titleFont = document.getElementById('select-font-title').value;
      const highlightFont = document.getElementById('select-font-highlight').value;

      document.documentElement.style.setProperty('--font-title-family', `'${titleFont}'`);
      document.documentElement.style.setProperty('--font-highlight-family', `'${highlightFont}'`);
    }

    function updateSocials() {
      const ig = document.getElementById('soc-ig').value.trim();
      const be = document.getElementById('soc-be').value.trim();
      const yt = document.getElementById('soc-yt').value.trim();
      const fb = document.getElementById('soc-fb').value.trim();
      
      const container = document.getElementById('social-container');
      container.innerHTML = '';
      const colorAccent = document.getElementById('color-accent').value;

      if(ig) container.innerHTML += `<a href="https://instagram.com/${ig.replace('@','')}" target="_blank" class="font-body outline-none p-1 rounded font-bold" style="color:${colorAccent}"><i class="fa-brands fa-instagram text-lg"></i> ${ig}</a>`;
      if(be) container.innerHTML += `<a href="https://${be}" target="_blank" class="font-body outline-none p-1 rounded font-bold" style="color:${colorAccent}"><i class="fa-brands fa-behance text-lg"></i></a>`;
      if(yt) container.innerHTML += `<a href="https://${yt}" target="_blank" class="font-body outline-none p-1 rounded font-bold" style="color:${colorAccent}"><i class="fa-brands fa-youtube text-lg"></i></a>`;
      if(fb) container.innerHTML += `<a href="https://${fb}" target="_blank" class="font-body outline-none p-1 rounded font-bold" style="color:${colorAccent}"><i class="fa-brands fa-facebook text-lg"></i></a>`;
    }

    function toggleSection(sectionId, isVisible) {
      const section = document.getElementById(sectionId);
      if (isVisible) section.classList.remove('hidden');
      else section.classList.add('hidden');
    }

    // --- IMAGENS E ARQUIVOS ---
    function loadProfileImage(event) {
      const reader = new FileReader();
      reader.onload = function() { document.getElementById('profile-img').src = reader.result; }
      if(event.target.files[0]) reader.readAsDataURL(event.target.files[0]);
    }

    function loadPortfolioImage(event, id) {
      const reader = new FileReader();
      reader.onload = function() { 
        const img = document.getElementById('port-img-' + id);
        img.src = reader.result;
        img.classList.remove('hidden');
      }
      if(event.target.files[0]) reader.readAsDataURL(event.target.files[0]);
    }

    function updatePortfolioLink(id, url) {
      let finalUrl = url;
      if (url && !url.startsWith('http://') && !url.startsWith('https://')) {
        finalUrl = 'https://' + url;
      }
      document.getElementById('port-link-' + id).href = finalUrl || '#';
    }

    // --- PRESETS ---
    function applyPreset(type) {
      if (type === 'neon') {
        document.getElementById('color-bg').value = '#0d0f17';
        document.getElementById('color-card').value = '#161b26';
        document.getElementById('color-accent').value = '#00f2fe';
        document.getElementById('color-border').value = '#334155';
      } else if (type === 'dark') {
        document.getElementById('color-bg').value = '#050508';
        document.getElementById('color-card').value = '#111118';
        document.getElementById('color-accent').value = '#ec4899';
        document.getElementById('color-border').value = '#27272a';
      } else if (type === 'clean') {
        document.getElementById('color-bg').value = '#f8fafc';
        document.getElementById('color-card').value = '#ffffff';
        document.getElementById('color-accent').value = '#059669';
        document.getElementById('color-border').value = '#e2e8f0';
      }
      updateGlobalColors();
    }

    window.onload = () => {
      updateGlobalColors();
      updateFontFamilies();
    };
  </script>
</body>
</html>

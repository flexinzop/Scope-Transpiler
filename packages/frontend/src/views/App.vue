<script setup lang="ts">
import { ref } from 'vue';
import { useSDK } from '../plugins/sdk';

interface HackerOneAsset {
  host: string;
  type: string;
}

interface HackerOneScope {
  target?: {
    scope?: {
      include?: HackerOneAsset[];
      exclude?: HackerOneAsset[];
    }
  }
}

const sdk = useSDK();
const inScope = ref('');
const outScope = ref('');

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];

  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    try {
      const content = e.target?.result as string;
      const json = JSON.parse(content);
      processScope(json);
    } catch (err) {
      sdk.window.showToast("Erro processing JSON file", { variant: "error" });
    }
  };
  reader.readAsText(file);
};

const inScopeCount = ref(0);
const outScopeCount = ref(0);

const processScope = (data: HackerOneScope) => {
  const clean = (host: string) => host.replace(/^\^/, '').replace(/\$$/, '').replace(/\\\./g, '.').replace(/\.\*/g, '*');

  // Usando nullish coalescing (??) para evitar erros de expressão booleana estrita
  const includes = data.target?.scope?.include ?? [];
  const excludes = data.target?.scope?.exclude ?? [];

  // Tipando explicitamente os itens do map
  const cleanedIncludes = includes.map((i: HackerOneAsset) => clean(i.host));
  const cleanedExcludes = excludes.map((e: HackerOneAsset) => clean(e.host));

  inScope.value = Array.from(new Set(cleanedIncludes)).join('\n');
  outScope.value = Array.from(new Set(cleanedExcludes)).join('\n');
  
  // Atualiza contadores (se você os incluiu)
  inScopeCount.value = cleanedIncludes.length;
  outScopeCount.value = cleanedExcludes.length;

  sdk.window.showToast("Arquivo processado com sucesso!", { variant: "success" });
};

const presetName = ref('H1 Program Scope'); // Default Name (placeholder)

const createPreset = async () => {
  if (!inScope.value && !outScope.value) {
    sdk.window.showToast("No data to import", { variant: "error" });
    return;
  }

  try {
    // We convert strings back to Arrays, removing empty rows
    const allowlist = inScope.value.split('\n').filter(line => line.trim() !== '');
    const denylist = outScope.value.split('\n').filter(line => line.trim() !== '');

    const scope = await sdk.scopes.createScope({
      name: presetName.value,
      allowlist: allowlist,
      denylist: denylist,
    });

    if (scope !== undefined) {
      sdk.window.showToast(`Created "${scope.name}" preset successfully!`, { variant: "success" });
    }
  } catch (err) {
    sdk.window.showToast("Error creating Caido Scope preset", { variant: "error" });
    console.error(err);
  }
};

</script>

<template>
  <div class="flex flex-col h-[calc(100vh-40px)] gap-4 p-6 bg-[#30333b] text-[#e0e0e0] overflow-hidden font-sans">
    <header class="flex justify-between items-center border-b border-[#333] pb-4">
      <h1 class="text-2xl font-bold tracking-tight">Scope Transpiler</h1>
      <div class="flex gap-2">
         <input 
          type="file" 
          id="file-upload"
          @change="handleFileSelect" 
          accept=".json"
          class="hidden"
        />
        <label for="file-upload" class="cursor-pointer bg-[#40b363] hover:bg-[#296e3d] border border-[#444] px-4 py-2 rounded text-sm transition-all">
          Choose H1 JSON
        </label>
      </div>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <div class="bg-[#343840] border-l-4 border-[#3b82f6] p-4 rounded-md shadow-lg">
        <p class="text-[10px] uppercase font-bold text-gray-400 tracking-widest">Included Hosts</p>
        <p class="text-4xl font-mono mt-1 text-white">{{ inScopeCount }}</p>
      </div>
      
      <div class="bg-[#343840] border-l-4 border-[#f97316] p-4 rounded-md shadow-lg">
        <p class="text-[10px] uppercase font-bold text-gray-400 tracking-widest">Excluded Hosts</p>
        <p class="text-4xl font-mono mt-1 text-white">{{ outScopeCount }}</p>
      </div>
    </div>

    <div class="flex gap-3 items-center bg-[#343840] p-3 rounded-md border border-[#2e2e2e]">
      <div class="flex-1 flex items-center bg-[#343840] border border-[#333] rounded px-3 focus-within:border-[#3b82f6] transition-all">
        <span class="text-xs text-gray-500 font-bold mr-2 uppercase">Name:</span>
        <input 
          v-model="presetName" 
          type="text" 
          class="flex-1 bg-transparent py-2 text-sm text-gray-200 outline-none"
        />
      </div>
      <button 
        @click="createPreset" 
        class="bg-[#3b82f6] hover:bg-[#2563eb] text-white text-sm font-bold py-2 px-6 rounded transition-all shadow-md active:scale-95"
      >
        Create Preset
      </button>
    </div>

    <div class="grid grid-cols-2 gap-6 flex-1">
      <section class="flex flex-col gap-2">
        <label class="text-sm font-bold text-gray-400">In Scope (Glob)</label>
        <textarea v-model="inScope" readonly class="flex-1 w-full bg-[#272a30] border border-[#333] p-3 rounded font-mono text-sm focus:outline-none"></textarea>
      </section>
      <section class="flex flex-col gap-2">
        <label class="text-sm font-bold text-gray-400">Out of Scope (Glob)</label>
        <textarea v-model="outScope" readonly class="flex-1 w-full bg-[#272a30] border border-[#333] p-3 rounded font-mono text-sm focus:outline-none"></textarea>
      </section>
    </div>
  </div>
</template>
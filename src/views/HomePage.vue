<template>
  <ion-page class="dionysus-page">
    <!-- Boot Splash (#boot) with Rotating Ring & Staggered Wordmark -->
    <transition name="boot-fade">
      <div id="boot" v-if="showSplash">
        <div class="boot-container">
          <div class="mark-wrapper">
            <div class="spin-ring"></div>
            <div class="d-symbol font-brand">D</div>
          </div>
          <div class="wordmark-track font-brand">
            <span
              v-for="(char, idx) in 'DIONYSUS'"
              :key="idx"
              :style="{ animationDelay: `${idx * 0.08}s` }"
            >
              {{ char }}
            </span>
          </div>
          <p class="boot-sub font-mono-code">INITIALIZING CRYPTOGRAPHIC CORE...</p>
        </div>
      </div>
    </transition>

    <ion-content class="dionysus-content" :scroll-y="true">
      <div class="dionysus-shell">
        <!-- Header -->
        <header class="app-header">
          <div class="header-main-row">
            <div class="brand-cluster">
              <div class="brand-avatar font-brand">D</div>
              <div class="brand-titles">
                <h1 class="brand-name font-brand">DIONYSUS</h1>
                <p class="brand-tagline font-tech">Reversible Cryptographic Studio</p>
              </div>
            </div>

            <div class="header-actions">
              <button 
                id="sound-toggle-btn"
                type="button"
                @click="toggleSound()" 
                class="btn-tactile header-btn sound-btn"
                :title="state.soundEnabled ? 'Mute audio' : 'Enable audio'"
              >
                <svg v-if="state.soundEnabled" class="header-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.536 8.464a5 5 0 010 7.072m2.828-9.9a9 9 0 010 12.728M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15z"/>
                </svg>
                <svg v-else class="header-icon text-muted" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5.586 15H4a1 1 0 01-1-1v-4a1 1 0 011-1h1.586l4.707-4.707C10.923 3.663 12 4.109 12 5v14c0 .891-1.077 1.337-1.707.707L5.586 15zM17 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2"/>
                </svg>
                <span class="header-sound-text font-tech">{{ state.soundEnabled ? 'Audio' : 'Muted' }}</span>
              </button>

              <button 
                id="drawer-toggle-btn"
                type="button"
                @click="toggleInspectorDrawer()" 
                class="btn-tactile header-btn shifts-btn font-tech"
                title="Inspect All 25 Shifts"
              >
                <svg class="header-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h7"/>
                </svg>
                <span class="shifts-text font-tech">25 Shifts</span>
              </button>

              <button 
                id="sample-preset-btn"
                type="button"
                @click="loadPresetSample()" 
                class="btn-tactile header-btn sample-pill font-tech"
              >
                Sample
              </button>
            </div>
          </div>
        </header>

        <!-- Tabs -->
        <nav class="view-tabs font-tech" role="tablist">
          <div 
            class="view-tabs-pill" 
            :style="{ transform: state.activeTab === 'methods' ? 'translateX(100%)' : 'translateX(0%)' }"
            :class="{ 'is-parked': state.activeTab === 'alignment' }"
          ></div>
          <button
            type="button"
            class="view-tab"
            :class="{ active: state.activeTab === 'cipher' || state.activeTab === 'alignment' }"
            @click="setActiveTab('cipher')"
          >Cipher</button>
          <button
            type="button"
            class="view-tab"
            :class="{ active: state.activeTab === 'methods' }"
            @click="setActiveTab('methods')"
          >Methods</button>
        </nav>

        <!-- Viewport -->
        <div class="tabs-viewport">
          <Transition :name="tabTransition" mode="out-in">
            <!-- TAB 1: Studio -->
            <main
              v-if="state.activeTab === 'cipher'"
              id="panel-cipher"
              class="studio-grid"
              key="tab-cipher"
            >
              <!-- Config Panel -->
              <section class="studio-col">
                <div class="studio-panel font-tech" id="cipher-config-panel">
                  <div class="panel-header">
                    <h2 class="panel-title font-brand">Cipher Configuration</h2>
                    <span class="method-tag font-mono-code">{{ METHOD_UI[state.method]?.desc || 'Uniform Shift' }}</span>
                  </div>

                  <div class="control-group">
                    <label class="section-label">Algorithm</label>
                    <div class="segmented-track" :class="{ 'is-muted': !FEATURED_METHODS.includes(state.method) }">
                      <div 
                        class="pill-slider" 
                        :style="{ 
                          transform: state.method === 'vigenere' ? 'translateX(100%)' : 'translateX(0%)',
                          opacity: FEATURED_METHODS.includes(state.method) ? '1' : '0' 
                        }"
                      ></div>
                      <button 
                        type="button" 
                        @click="setMethod('caesar')" 
                        class="segmented-btn"
                        :class="{ active: state.method === 'caesar' }"
                      >
                        Caesar Shift
                      </button>
                      <button 
                        type="button" 
                        @click="setMethod('vigenere')" 
                        class="segmented-btn"
                        :class="{ active: state.method === 'vigenere' }"
                      >
                        Vigenère Cipher
                      </button>
                    </div>
                  </div>

                  <div class="control-group">
                    <label class="section-label">Operation</label>
                    <div class="segmented-track">
                      <div 
                        class="pill-slider" 
                        :style="{ transform: state.action === 'decrypt' ? 'translateX(100%)' : 'translateX(0%)' }"
                      ></div>
                      <button 
                        type="button" 
                        @click="setAction('encrypt')" 
                        class="segmented-btn"
                        :class="{ active: state.action === 'encrypt' }"
                      >
                        <svg class="control-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
                        </svg>
                        <span>Encrypt</span>
                      </button>
                      <button 
                        type="button" 
                        @click="setAction('decrypt')" 
                        class="segmented-btn"
                        :class="{ active: state.action === 'decrypt' }"
                      >
                        <svg class="control-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 11V7a4 4 0 118 0m-4 8v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z"/>
                        </svg>
                        <span>Decrypt</span>
                      </button>
                    </div>
                  </div>

                  <div class="control-group">
                    <label class="section-label">All Methods</label>
                    <button
                      type="button"
                      class="btn-tactile method-picker-btn"
                      @click="openMethodsModal()"
                    >
                      <div class="method-picker-stack">
                        <span class="picker-prefix font-mono-code">USING</span>
                        <span class="method-picker-name font-brand">{{ currentMethodDetails?.name || 'Caesar Shift' }}</span>
                      </div>
                      <div class="method-picker-action">
                        <span class="action-text">Browse all</span>
                        <svg class="chevron-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 9l7 7 7-7"/>
                        </svg>
                      </div>
                    </button>
                    <p class="picker-family-desc font-mono-code">{{ currentMethodDetails?.family || 'Monoalphabetic Substitution' }}</p>
                  </div>

                  <div class="control-group key-config-group">
                    <div class="row-between">
                      <label class="section-label">{{ METHOD_UI[state.method]?.label || 'Key' }}</label>
                      <span class="key-hint font-mono-code">{{ METHOD_UI[state.method]?.hint() }}</span>
                    </div>

                    <p v-if="state.method === 'atbash'" class="no-key-note font-mono-code">
                      Atbash needs no key — it reflects each letter across the middle of the alphabet (A ↔ Z, B ↔ Y, C ↔ X).
                    </p>

                    <div v-if="state.method === 'affine'" class="affine-grid">
                      <div class="input-subgroup">
                        <label class="sub-label">Multiplier</label>
                        <div class="select-wrapper">
                          <select v-model.number="state.affineMul" class="field-input select-styled font-mono-code" @change="runTransformation(true)">
                            <option v-for="m in AFFINE_MULTIPLIERS" :key="m" :value="m">{{ m }}</option>
                          </select>
                          <svg class="select-chevron" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 9l6 6 6-6"/>
                          </svg>
                        </div>
                      </div>
                      <div class="input-subgroup">
                        <label class="sub-label">Offset</label>
                        <input
                          type="number"
                          v-model.number="state.affineOffset"
                          min="0"
                          max="25"
                          class="field-input font-mono-code"
                          @input="runTransformation(true)"
                        />
                      </div>
                    </div>

                    <div v-if="state.method === 'railfence'" class="affine-grid">
                      <div class="input-subgroup">
                        <label class="sub-label">Rails Depth</label>
                        <div class="select-wrapper">
                          <select v-model.number="state.railFenceDepth" class="field-input select-styled font-mono-code" @change="runTransformation(true)">
                            <option v-for="n in [2,3,4,5,6,7,8]" :key="n" :value="n">{{ n }}</option>
                          </select>
                          <svg class="select-chevron" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 9l6 6 6-6"/>
                          </svg>
                        </div>
                      </div>
                    </div>

                    <div v-if="METHOD_UI[state.method]?.kind === 'number' || METHOD_UI[state.method]?.kind === 'text'" class="key-input-row">
                      <template v-if="state.method === 'caesar'">
                        <input 
                          type="number" 
                          v-model.number="state.caesarShift" 
                          min="0" 
                          max="25" 
                          @input="handleKeyChange($event.target.value)"
                          class="field-input num-field font-mono-code"
                        />
                        <div class="stepper-cluster">
                          <button type="button" @click="adjustShift(-1)" class="btn-tactile step-btn">-1</button>
                          <button type="button" @click="adjustShift(1)" class="btn-tactile step-btn">+1</button>
                          <button type="button" @click="applyRot13()" class="btn-tactile step-btn highlight-btn">ROT13</button>
                          <button type="button" @click="randomizeKey()" class="btn-tactile step-btn">Rand</button>
                        </div>
                      </template>

                      <template v-else-if="state.method === 'vigenere'">
                        <input 
                          type="text" 
                          v-model="state.vigenereWord" 
                          placeholder="BACCHUS"
                          @input="handleKeyChange($event.target.value)"
                          class="field-input text-field font-mono-code uppercase"
                        />
                      </template>

                      <template v-else-if="state.method === 'keyword'">
                        <input 
                          type="text" 
                          v-model="state.keyword" 
                          placeholder="ZEBRA"
                          @input="handleKeyChange($event.target.value)"
                          class="field-input text-field font-mono-code uppercase"
                        />
                      </template>
                    </div>

                    <div v-if="state.method === 'caesar'" class="slider-box">
                      <input 
                        type="range" 
                        min="0" 
                        max="25" 
                        v-model.number="state.caesarShift" 
                        @input="handleSliderChange($event.target.value)"
                      />
                    </div>
                  </div>
                </div>
              </section>

              <!-- Output & IO Panel -->
              <section class="studio-col">
                <div class="studio-panel font-tech">
                  <div class="control-group">
                    <div class="row-between">
                      <label class="section-label">
                        {{ state.action === 'encrypt' ? 'Plaintext Message' : 'Ciphertext Message' }}
                      </label>
                      <span class="char-count font-mono-code">{{ messageText.length }} characters</span>
                    </div>
                    <textarea 
                      v-model="messageText" 
                      rows="3" 
                      placeholder="Enter message to transform..."
                      @input="onMessageInput"
                      class="field-input textarea-field font-mono-code"
                    ></textarea>
                  </div>

                  <div class="control-group">
                    <div class="row-between output-header-row">
                      <div class="output-label-box">
                        <label class="section-label">
                          {{ state.action === 'encrypt' ? 'Encrypted Result' : 'Decrypted Result' }}
                        </label>
                        <span class="format-badge font-mono-code">{{ state.spacingMode.toUpperCase() }}</span>
                      </div>

                      <div class="output-actions-box">
                        <button 
                          type="button" 
                          @click="cycleSpacingMode()" 
                          class="btn-tactile pill-btn font-mono-code"
                        >
                          Mode: {{ state.spacingMode === 'spaced' ? 'Spaced' : state.spacingMode === 'block5' ? '5-Block' : 'Compact' }}
                        </button>

                        <button 
                          type="button" 
                          @click="copyResult()" 
                          class="btn-tactile copy-action-btn font-mono-code"
                        >
                          <svg class="btn-inline-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"/>
                          </svg>
                          <span>{{ copyBtnText }}</span>
                        </button>
                      </div>
                    </div>

                    <div class="card-surface output-canvas">
                      <div class="canvas-top-telemetry">
                        <span class="telemetry-tag font-mono-code" :class="{ 'is-active': isAnimating }">
                          {{ isAnimating ? 'STREAM DECRYPTING...' : 'SYNCED' }}
                        </span>
                      </div>

                      <div 
                        class="canvas-text font-mono-code"
                        :class="{ 'spaced-char-stream': state.spacingMode === 'spaced' }"
                      >
                        <span 
                          v-for="(t, idx) in streamTokens" 
                          :key="idx"
                          class="stream-char"
                          :class="{ 'is-scrambling': t.scrambling }"
                        >{{ t.char }}</span>
                        <span class="blinking-cursor"></span>
                      </div>

                      <div class="canvas-raw-preview font-mono-code">
                        Raw Stream: {{ state.lastRawResult || '-' }}
                      </div>
                    </div>
                  </div>

                  <div class="bottom-actions-row">
                    <button 
                      type="button" 
                      @click="swapResultToInput()" 
                      class="btn-tactile bottom-btn swap-action-btn"
                    >
                      <svg class="btn-inline-icon rotator" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16V4m0 0L3 8m4-4l4 4m6 0v12m0 0l4-4m-4 4l-4-4"/>
                      </svg>
                      <span>Use Result as Input</span>
                    </button>

                    <button 
                      type="button" 
                      @click="downloadTextPayload()" 
                      class="btn-tactile bottom-btn"
                    >
                      <svg class="btn-inline-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"/>
                      </svg>
                      <span>Save .txt</span>
                    </button>

                    <button 
                      type="button" 
                      @click="clearAllFields()" 
                      class="btn-tactile bottom-btn clear-action-btn"
                    >
                      Clear
                    </button>
                  </div>
                </div>
              </section>
            </main>

            <!-- TAB 2: Methods Compendium (Compact & Clean Layout) -->
            <main
              v-else-if="state.activeTab === 'methods'"
              id="panel-methods"
              class="methods-container"
              key="tab-methods"
            >
              <section class="studio-panel intro-panel font-tech">
                <h2 class="panel-title font-brand">Cipher Compendium</h2>
                <p class="compendium-sub">
                  Six reversible ciphers, from a fixed mirror to a keyword-seeded alphabet.
                  Each card covers how the mapping is built, what it buys you, and the weakness that gives it away.
                </p>
              </section>

              <div class="method-card-grid">
                <article
                  v-for="m in METHOD_LIBRARY"
                  :key="m.id"
                  class="method-card font-tech"
                  :class="{ 'is-active': state.method === m.id }"
                >
                  <header class="method-card-head">
                    <div class="method-title-group">
                      <h3 class="method-card-name font-brand">{{ m.name }}</h3>
                      <div class="method-meta-line font-mono-code">
                        <span>[{{ m.family.toUpperCase() }}]</span>
                        <span class="meta-dot">·</span>
                        <span>{{ m.era }}</span>
                      </div>
                    </div>
                    <span v-if="m.id === state.method" class="method-card-badge font-mono-code">ACTIVE</span>
                  </header>

                  <p class="method-card-summary font-tech">{{ m.summary }}</p>

                  <div class="method-code-box">
                    <div class="method-card-formula font-mono-code">{{ m.formula }}</div>
                    <div class="method-card-example font-mono-code">{{ m.example }}</div>
                  </div>

                  <dl class="method-card-notes">
                    <div class="note-row">
                      <span class="note-label font-mono-code">PROS:</span>
                      <span class="note-val">{{ m.strength }}</span>
                    </div>
                    <div class="note-row">
                      <span class="note-label font-mono-code">CONS:</span>
                      <span class="note-val">{{ m.caveat }}</span>
                    </div>
                  </dl>

                  <button
                    type="button"
                    class="method-card-use btn-tactile"
                    :disabled="state.method === m.id"
                    @click="useMethodFromOverview(m.id)"
                  >
                    <span>{{ state.method === m.id ? 'Currently selected' : 'Use this method' }}</span>
                    <svg v-if="state.method !== m.id" class="card-arrow-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14m0 0l-5-5m5 5l-5 5"/>
                    </svg>
                  </button>
                </article>
              </div>
            </main>

            <!-- TAB 3: Alignment View -->
            <section
              v-else-if="state.activeTab === 'alignment'"
              id="panel-alignment"
              class="studio-panel alignment-panel font-tech"
              key="tab-alignment"
            >
              <div class="panel-header">
                <h2 class="panel-title font-brand">Alphabet Alignment</h2>
                <span class="entropy-badge font-mono-code">Entropy: {{ entropyScore.toFixed(2) }} ({{ entropyLabel }})</span>
              </div>

              <p class="alignment-caption font-mono-code">
                Live tape for <span class="active-cipher-name">{{ cipherLabel }}</span>
                <span class="text-dim"> · </span>
                <span>Plaintext mapping against transformed output</span>
              </p>

              <div class="tape-viewport">
                <div class="tape-title-row">
                  <span class="status-dot"></span>
                  <span class="tape-title-text font-mono-code">
                    {{ METHOD_UI[state.method]?.ribbon || 'Alphabet Alignment' }}
                  </span>
                </div>

                <div class="alignment-ribbon">
                  <div 
                    v-for="(cell, i) in ribbonCells" 
                    :key="i"
                    class="ribbon-cell"
                    :class="{ 'shifted-highlight': cell.active }"
                  >
                    <span class="cell-top font-mono-code">{{ cell.top }}</span>
                    <span class="cell-bot font-mono-code">{{ cell.bot }}</span>
                  </div>
                </div>
              </div>
            </section>
          </Transition>
        </div>
      </div>
    </ion-content>

    <!-- Bottom CTAs -->
    <div class="dionysus-cta" v-if="state.activeTab === 'cipher'">
      <div class="dionysus-cta-inner">
        <button
          type="button"
          class="dionysus-cta-btn font-tech"
          @click="setActiveTab('alignment')"
        >
          <svg class="cta-left-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h10M4 18h13"/>
          </svg>
          <span class="cta-btn-text">Alphabet Alignment</span>
          <svg class="cta-arrow-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14m0 0l-5-5m5 5l-5 5"/>
          </svg>
        </button>
        <p class="dionysus-cta-hint font-mono-code">See how each letter maps to its encrypted position</p>
      </div>
    </div>

    <div class="dionysus-cta" v-if="state.activeTab === 'alignment'">
      <div class="dionysus-cta-inner">
        <button
          type="button"
          class="dionysus-cta-btn font-tech"
          @click="setActiveTab('cipher')"
        >
          <svg class="cta-left-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 12H5m0 0l5-5m-5 5l5 5"/>
          </svg>
          <span class="cta-btn-text">Back to Cipher</span>
          <svg class="cta-arrow-icon rotate-180" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14m0 0l-5-5m5 5l-5 5"/>
          </svg>
        </button>
        <p class="dionysus-cta-hint font-mono-code">{{ cipherLabel }}</p>
      </div>
    </div>

    <!-- Centered Dialog Modal -->
    <Transition name="dialog-pop">
      <div
        v-if="state.modalOpen"
        class="modal-overlay"
        @click.self="closeMethodsModal()"
      >
        <div class="modal-dialog-panel" role="document">
          <header class="modal-head">
            <div class="truncate">
              <h2 class="font-brand modal-title">Select a Cipher</h2>
              <p class="modal-sub font-tech">Caesar and Vigenère are the fast path · six total</p>
            </div>
            <button
              type="button"
              class="modal-close btn-tactile"
              @click="closeMethodsModal()"
            >
              <svg fill="none" viewBox="0 0 24 24" stroke="currentColor" class="w-3.5 h-3.5">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 6l12 12M18 6L6 18"/>
              </svg>
            </button>
          </header>

          <div class="modal-list">
            <button
              v-for="m in METHOD_LIBRARY"
              :key="m.id"
              type="button"
              class="method-option font-tech"
              :class="{ 'is-active': state.method === m.id }"
              @click="setMethod(m.id); closeMethodsModal()"
            >
              <span class="method-option-tick font-mono-code">
                <svg fill="none" viewBox="0 0 24 24" stroke="currentColor" class="w-3 h-3">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
                </svg>
              </span>
              <div class="truncate flex-1 text-left">
                <span class="method-option-name font-brand">{{ m.name }}</span>
                <span class="method-option-family font-mono-code">[{{ m.family.toUpperCase() }}]</span>
                <span class="method-option-summary">{{ m.summary }}</span>
              </div>
              <span v-if="m.id === 'caesar' || m.id === 'vigenere'" class="method-option-flag font-mono-code">Featured</span>
            </button>
          </div>

          <footer class="modal-foot">
            <button
              type="button"
              class="btn-tactile full-readmore-btn font-tech"
              @click="closeMethodsModal(); setActiveTab('methods')"
            >
              <span>Read the full breakdown</span>
              <svg class="readmore-arrow-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 12h14m0 0l-5-5m5 5l-5 5"/>
              </svg>
            </button>
          </footer>
        </div>
      </div>
    </Transition>

    <!-- All-Shifts Drawer -->
    <div 
      id="inspector-overlay" 
      class="drawer-overlay" 
      :class="{ active: drawerOpen }"
      @click="drawerOpen = false"
    >
      <div class="drawer-panel" @click.stop>
        <div class="drawer-head">
          <div class="truncate mr-2">
            <h3 class="font-brand font-bold text-sm text-[#f5f5f5]">All 25 Caesar Shifts</h3>
            <p class="text-[11px] text-[#a3a3a3] font-tech">Brute-force table with current text</p>
          </div>
          <button 
            type="button" 
            @click="drawerOpen = false"
            class="drawer-close-btn btn-tactile font-tech"
          >
            Close
          </button>
        </div>

        <div class="drawer-body font-mono-code">
          <div
            v-for="s in 25"
            :key="s"
            class="brute-card"
            @click="applyShift(s)"
          >
            <div class="row-between text-[11px]">
              <span class="font-bold text-[#f5f5f5]">Shift +{{ s }} ({{ String.fromCharCode(65 + s) }})</span>
              <span class="text-[10px] text-[#ffffff]">Tap to apply</span>
            </div>
            <div class="brute-text">
              {{ getShiftPreview(s) }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Slim & Exact Centered Floating Toast Notification -->
    <div 
      class="toast-pill font-mono-code"
      :class="{ 'show': toastVisible }"
    >
      <svg class="toast-check-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
      </svg>
      <span class="toast-content">{{ toastMsg }}</span>
    </div>
  </ion-page>
</template>

<script setup>
import { onBeforeUnmount, onMounted, reactive, computed, ref } from 'vue';
import { IonPage, IonContent } from '@ionic/vue';

const showSplash = ref(true);

const state = reactive({
  method: 'caesar',
  action: 'encrypt',
  activeTab: 'cipher',
  caesarShift: 3,
  vigenereWord: 'BACCHUS',
  keyword: 'ZEBRA',
  affineMul: 5,
  affineOffset: 3,
  railFenceDepth: 3,
  soundEnabled: true,
  spacingMode: 'spaced',
  lastRawResult: '',
  modalOpen: false
});

const tabTransition = ref('tab-fade');
const messageText = ref('Hello Dionysus');
const copyBtnText = ref('Copy');
const isAnimating = ref(false);
const streamTokens = ref([]);
const drawerOpen = ref(false);
const toastVisible = ref(false);
const toastMsg = ref('');
let toastTimer = null;
let scrambleInterval = null;

const FEATURED_METHODS = ['caesar', 'vigenere'];
const AFFINE_MULTIPLIERS = [5, 7, 9, 11, 15, 17, 19, 21, 23, 25];

const METHOD_LIBRARY = [
  {
    id: 'caesar',
    name: 'Caesar Shift',
    family: 'Monoalphabetic Substitution',
    era: 'Ancient Rome, c. 58 BC',
    formula: 'C = (P + k) mod 26',
    summary: 'Every letter slides the same number of places along the alphabet. One number is the entire key.',
    example: 'k = 3 · HELLO → KHOOR',
    strength: 'Instant to read, learn and hand-signal.',
    caveat: 'Only 25 meaningful keys. Real ciphertext falls in under a second of brute force.',
  },
  {
    id: 'vigenere',
    name: 'Vigenère Cipher',
    family: 'Polyalphabetic Substitution',
    era: 'Bellaso 1553 · named 1586',
    formula: 'C = (P + Kᵢ) mod 26',
    summary: 'A keyword repeats under the message and each position is shifted by its own key letter.',
    example: 'key LEMON · ATTACKATDAWN → LXFOPVEFRNHR',
    strength: 'Breaks single-substitution frequency analysis.',
    caveat: 'Repeating keyword length leaks over Kasiski analysis.',
  },
  {
    id: 'atbash',
    name: 'Atbash',
    family: 'Monoalphabetic Substitution',
    era: 'c. 600 BC, Hebrew',
    formula: 'C = 25 − P',
    summary: 'The alphabet folded onto itself: A trades places with Z, B with Y, and so on.',
    example: 'HELLO → SVOOL',
    strength: 'Nothing to memorise, and decryption is the identical inverse.',
    caveat: 'A single fixed table easily broken by frequency matching.',
  },
  {
    id: 'keyword',
    name: 'Keyword Substitution',
    family: 'Keyed Monoalphabetic',
    era: '19th-century cipher manuals',
    formula: 'alphabet = unique keyword letters + unused A–Z',
    summary: 'Builds a scrambled alphabet starting with unique keyword letters.',
    example: 'ZEBRA → ZEBRACDFGHIJKLMNOPQSTUVWXY',
    strength: 'Generates a full 26-letter permutation from a single word.',
    caveat: 'Plaintext letter frequencies survive intact.',
  },
  {
    id: 'affine',
    name: 'Affine',
    family: 'Linear Polyalphabetic',
    era: 'Classical Greek mathematics',
    formula: 'C = (a·P + b) mod 26',
    summary: 'Scales the letter value before shifting it via coprime multipliers.',
    example: 'a = 5, b = 3 · HELLO → MXGGV',
    strength: 'Larger keyspace from a simple formula.',
    caveat: 'Still monoalphabetic in structure.',
  },
  {
    id: 'railfence',
    name: 'Rail Fence',
    family: 'Transposition',
    era: '19th century',
    formula: 'zigzag writing over N rails',
    summary: 'The letters survive unchanged in identity but are shuffled in sequence.',
    example: 'depth 3 · ABCDEFGH → AEBDFHCG',
    strength: 'Preserves exact frequencies while scrambling order.',
    caveat: 'Vulnerable to geometric pattern matching and anagramming.',
  },
];

const METHOD_UI = {
  caesar: {
    label: 'Shift Number (Key)',
    desc: 'Uniform Shift',
    ribbon: 'Alphabet Alignment (A → Shifted)',
    kind: 'number',
    hint: () => `Shift: +${state.caesarShift} (${String.fromCharCode(65 + (state.caesarShift % 26))})`,
  },
  vigenere: {
    label: 'Secret Keyword',
    desc: 'Polyalphabetic',
    ribbon: 'Keystream Alignment',
    kind: 'text',
    hint: () => `Key "${state.vigenereWord || 'KEY'}"`,
  },
  atbash: {
    label: 'Key',
    desc: 'Mirror Alphabet',
    ribbon: 'A ↔ Z Reflection',
    kind: 'none',
    hint: () => 'No key needed — the alphabet mirrors itself',
  },
  keyword: {
    label: 'Substitution Keyword',
    desc: 'Keyed Alphabet',
    ribbon: 'Keyed Alphabet Mapping',
    kind: 'text',
    hint: () => `Key "${state.keyword || 'KEY'}"`,
  },
  affine: {
    label: 'Multiplier & Offset',
    desc: 'Linear Formula',
    ribbon: 'Multiplicative Mapping',
    kind: 'affine',
    hint: () => `E(x) = ${state.affineMul}x + ${state.affineOffset} (mod 26)`,
  },
  railfence: {
    label: 'Rail Depth',
    desc: 'Transposition',
    ribbon: 'Zigzag Rail Layout',
    kind: 'rails',
    hint: () => `${state.railFenceDepth} rails — reordered`,
  },
};

const currentMethodDetails = computed(() => {
  return METHOD_LIBRARY.find(m => m.id === state.method);
});

const cipherLabel = computed(() => {
  if (state.method === 'caesar') {
    return `Caesar Shift +${state.caesarShift} (${String.fromCharCode(65 + (state.caesarShift % 26))})`;
  }
  if (state.method === 'vigenere') {
    return `Vigenère "${state.vigenereWord || 'KEY'}"`;
  }
  return currentMethodDetails.value?.name || state.method;
});

// Audio Web Synthesizer
let audioCtx = null;
function playTactileFeedback(freq = 680, duration = 0.016) {
  if (!state.soundEnabled) return;
  try {
    if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    if (audioCtx.state === 'suspended') audioCtx.resume();
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.type = 'triangle';
    osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
    gain.gain.setValueAtTime(0.025, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.start();
    osc.stop(audioCtx.currentTime + duration);
  } catch (e) {}
}

function toggleSound() {
  state.soundEnabled = !state.soundEnabled;
  if (state.soundEnabled) {
    playTactileFeedback(840, 0.02);
    showToast('Tactile audio enabled');
  } else {
    showToast('Audio muted');
  }
}

// Cryptography Engine
function cipherCaesar(text, shift, decrypt = false) {
  const s = decrypt ? (26 - (shift % 26)) % 26 : (shift % 26 + 26) % 26;
  return text.replace(/[a-zA-Z]/g, (char) => {
    const code = char.charCodeAt(0);
    const base = code >= 65 && code <= 90 ? 65 : 97;
    return String.fromCharCode(((code - base + s) % 26) + base);
  });
}

function cipherVigenere(text, keyword, decrypt = false) {
  const cleanKey = (keyword || 'KEY').replace(/[^a-zA-Z]/g, '').toUpperCase();
  if (!cleanKey.length) return text;
  let keyIndex = 0;
  return text.split('').map((char) => {
    const code = char.charCodeAt(0);
    const isUpper = code >= 65 && code <= 90;
    const isLower = code >= 97 && code <= 122;
    if (!isUpper && !isLower) return char;
    const base = isUpper ? 65 : 97;
    const shiftVal = cleanKey.charCodeAt(keyIndex % cleanKey.length) - 65;
    const s = decrypt ? (26 - shiftVal) % 26 : shiftVal;
    keyIndex++;
    return String.fromCharCode(((code - base + s) % 26) + base);
  });
}

function cipherAtbash(text) {
  return text.replace(/[a-zA-Z]/g, (char) => {
    const code = char.charCodeAt(0);
    const base = code >= 65 && code <= 90 ? 65 : 97;
    return String.fromCharCode(base + 25 - (code - base));
  });
}

function buildKeywordAlphabet(keyword) {
  const clean = (keyword || '').replace(/[^a-zA-Z]/g, '').toUpperCase();
  const seen = new Set();
  let out = '';
  for (const ch of clean) {
    if (!seen.has(ch)) { seen.add(ch); out += ch; }
  }
  for (let i = 0; i < 26; i++) {
    const ch = String.fromCharCode(65 + i);
    if (!seen.has(ch)) { seen.add(ch); out += ch; }
  }
  return out;
}

function cipherKeyword(text, keyword, decrypt = false) {
  const alpha = buildKeywordAlphabet(keyword);
  const map = {};
  for (let i = 0; i < 26; i++) {
    const plain = String.fromCharCode(65 + i);
    const cipherCh = alpha[i];
    map[decrypt ? cipherCh : plain] = decrypt ? plain : cipherCh;
  }
  return text.replace(/[a-zA-Z]/g, (char) => {
    const upper = char.toUpperCase();
    const out = map[upper];
    if (!out) return char;
    return char === upper ? out : out.toLowerCase();
  });
}

function modInverse(a, m) {
  let [oldR, r] = [((a % m) + m) % m, m];
  let [oldS, s] = [1, 0];
  while (r !== 0) {
    const q = Math.floor(oldR / r);
    [oldR, r] = [r, oldR - q * r];
    [oldS, s] = [s, oldS - q * s];
  }
  return ((oldS % m) + m) % m;
}

function cipherAffine(text, mul, offset, decrypt = false) {
  const a = AFFINE_MULTIPLIERS.includes(mul) ? mul : 5;
  const b = (((offset % 26) + 26) % 26);
  const inv = modInverse(a, 26);
  const factor = decrypt ? inv : a;
  const shift = decrypt ? (((-inv * b) % 26) + 26) % 26 : b;

  return text.replace(/[a-zA-Z]/g, (char) => {
    const code = char.charCodeAt(0);
    const base = code >= 65 && code <= 90 ? 65 : 97;
    const x = code - base;
    return String.fromCharCode(((((factor * x + shift) % 26) + 26) % 26) + base);
  });
}

function cipherRailFence(text, depth, decrypt = false) {
  const chars = Array.from(text);
  const len = chars.length;
  if (len <= 1) return text;
  const d = Math.max(2, Math.min(depth, len));

  if (!decrypt) {
    const buckets = Array.from({ length: d }, () => []);
    let rail = 0, dir = 1;
    for (let i = 0; i < len; i++) {
      buckets[rail].push(chars[i]);
      if (rail === 0) dir = 1;
      else if (rail === d - 1) dir = -1;
      rail += dir;
    }
    return buckets.flat().join('');
  }

  const railsCount = new Array(d).fill(0);
  let rail = 0, dir = 1;
  for (let i = 0; i < len; i++) {
    railsCount[rail]++;
    if (rail === 0) dir = 1;
    else if (rail === d - 1) dir = -1;
    rail += dir;
  }

  const buckets = [];
  let idx = 0;
  for (let r = 0; r < d; r++) {
    buckets.push(chars.slice(idx, idx + railsCount[r]));
    idx += railsCount[r];
  }

  const out = new Array(len);
  rail = 0; dir = 1;
  for (let i = 0; i < len; i++) {
    out[i] = buckets[rail].shift();
    if (rail === 0) dir = 1;
    else if (rail === d - 1) dir = -1;
    rail += dir;
  }
  return out.join('');
}

function calculateShannonEntropy(str) {
  if (!str) return 0;
  const freq = {};
  for (let i = 0; i < str.length; i++) {
    freq[str[i]] = (freq[str[i]] || 0) + 1;
  }
  let entropy = 0;
  for (const ch in freq) {
    const p = freq[ch] / str.length;
    entropy -= p * Math.log2(p);
  }
  return entropy;
}

function formatWithSpaces(text, mode = state.spacingMode) {
  if (!text) return '';
  if (mode === 'spaced') return text.split('').join(' ');
  if (mode === 'block5') {
    const stripped = text.replace(/\s+/g, '');
    return stripped.match(/.{1,5}/g)?.join(' ') || text;
  }
  return text;
}

const entropyScore = computed(() => calculateShannonEntropy(state.lastRawResult));
const entropyLabel = computed(() => {
  if (entropyScore.value > 3.9) return 'High';
  if (entropyScore.value > 2.5) return 'Medium';
  return 'Low';
});

// Transformation & Scramble Loop
function runTransformation(animate = false) {
  const rawText = messageText.value;
  const isDecrypt = state.action === 'decrypt';
  let rawResult = '';

  if (state.method === 'caesar') {
    rawResult = cipherCaesar(rawText, state.caesarShift, isDecrypt);
  } else if (state.method === 'vigenere') {
    rawResult = cipherVigenere(rawText, state.vigenereWord, isDecrypt);
  } else if (state.method === 'atbash') {
    rawResult = cipherAtbash(rawText);
  } else if (state.method === 'keyword') {
    rawResult = cipherKeyword(rawText, state.keyword, isDecrypt);
  } else if (state.method === 'affine') {
    rawResult = cipherAffine(rawText, state.affineMul, state.affineOffset, isDecrypt);
  } else {
    rawResult = cipherRailFence(rawText, state.railFenceDepth, isDecrypt);
  }

  state.lastRawResult = rawResult;
  const formattedResult = formatWithSpaces(rawResult);

  if (!rawResult) {
    streamTokens.value = [{ char: 'Waiting for message input...', scrambling: false }];
    return;
  }

  if (animate) {
    triggerScrambleAnimation(formattedResult);
  } else {
    streamTokens.value = formattedResult.split('').map(c => ({ char: c, scrambling: false }));
  }
}

function triggerScrambleAnimation(targetString) {
  isAnimating.value = true;
  clearInterval(scrambleInterval);

  const glyphs = '0123456789ABCDEF!@#$%&*';
  const targetChars = targetString.split('');

  streamTokens.value = targetChars.map(char => ({
    char: char === ' ' || char === '\n' ? char : glyphs[Math.floor(Math.random() * glyphs.length)],
    scrambling: char !== ' ' && char !== '\n'
  }));

  let iteration = 0;
  scrambleInterval = setInterval(() => {
    streamTokens.value = targetChars.map((realChar, idx) => {
      if (idx < iteration || realChar === ' ' || realChar === '\n') {
        return { char: realChar, scrambling: false };
      }
      return {
        char: glyphs[Math.floor(Math.random() * glyphs.length)],
        scrambling: true
      };
    });

    if (iteration >= targetChars.length) {
      clearInterval(scrambleInterval);
      isAnimating.value = false;
    }
    iteration += 2;
  }, 16);
}

// Tape Calculation
const ribbonCells = computed(() => {
  const cells = [];
  if (state.method === 'caesar') {
    const s = state.action === 'decrypt' ? (26 - (state.caesarShift % 26)) % 26 : (state.caesarShift % 26);
    for (let i = 0; i < 26; i++) {
      cells.push({
        top: String.fromCharCode(65 + i),
        bot: String.fromCharCode(65 + ((i + s) % 26)),
        active: s !== 0
      });
    }
  } else if (state.method === 'atbash') {
    for (let i = 0; i < 26; i++) {
      cells.push({
        top: String.fromCharCode(65 + i),
        bot: String.fromCharCode(90 - i),
        active: true
      });
    }
  } else if (state.method === 'keyword') {
    const alpha = buildKeywordAlphabet(state.keyword);
    for (let i = 0; i < 26; i++) {
      cells.push({
        top: String.fromCharCode(65 + i),
        bot: alpha[i],
        active: alpha[i] !== String.fromCharCode(65 + i)
      });
    }
  } else if (state.method === 'affine') {
    for (let i = 0; i < 26; i++) {
      const mapped = cipherAffine(String.fromCharCode(65 + i), state.affineMul, state.affineOffset, state.action === 'decrypt');
      cells.push({
        top: String.fromCharCode(65 + i),
        bot: mapped,
        active: mapped !== String.fromCharCode(65 + i)
      });
    }
  } else if (state.method === 'railfence') {
    const d = Math.max(2, Math.min(state.railFenceDepth, 12));
    for (let r = 0; r < d; r++) {
      cells.push({ top: `R${r + 1}`, bot: 'rail', active: true });
    }
  } else {
    const key = (state.vigenereWord || 'KEY').replace(/[^a-zA-Z]/g, '').toUpperCase();
    const limit = Math.min(messageText.value.length || 6, 24);
    let keyIdx = 0;
    for (let i = 0; i < limit; i++) {
      const char = messageText.value[i] || 'A';
      const isLetter = /[a-zA-Z]/.test(char);
      const keyChar = isLetter ? key[keyIdx % key.length] : '-';
      if (isLetter) keyIdx++;
      const outChar = state.lastRawResult[i] || '';
      cells.push({
        top: keyChar,
        bot: outChar === ' ' ? '␣' : outChar,
        active: true
      });
    }
  }
  return cells;
});

// UI Actions
function setMethod(mode) {
  playTactileFeedback(640, 0.02);
  state.method = mode;
  runTransformation(true);
}

function setAction(action) {
  playTactileFeedback(680, 0.02);
  state.action = action;
  runTransformation(true);
}

function setActiveTab(tab) {
  state.activeTab = tab;
  playTactileFeedback(720, 0.016);
}

function cycleSpacingMode() {
  playTactileFeedback(760, 0.015);
  const modes = ['spaced', 'block5', 'compact'];
  state.spacingMode = modes[(modes.indexOf(state.spacingMode) + 1) % modes.length];
  runTransformation(true);
  showToast(`Spacing: ${state.spacingMode.toUpperCase()}`);
}

function handleKeyChange() {
  playTactileFeedback(740, 0.012);
  runTransformation(false);
}

function handleSliderChange(val) {
  playTactileFeedback(700 + (val * 8), 0.012);
  state.caesarShift = parseInt(val, 10);
  runTransformation(false);
}

function adjustShift(delta) {
  let next = (state.caesarShift + delta) % 26;
  if (next < 0) next += 26;
  state.caesarShift = next;
  playTactileFeedback(delta > 0 ? 820 : 660, 0.018);
  runTransformation(true);
}

function applyRot13() {
  state.caesarShift = 13;
  playTactileFeedback(900, 0.02);
  runTransformation(true);
  showToast('ROT13 applied (+13)');
}

function randomizeKey() {
  playTactileFeedback(940, 0.025);
  if (state.method === 'caesar') {
    state.caesarShift = Math.floor(Math.random() * 25) + 1;
  } else {
    const keys = ['BACCHUS', 'MYSTIC', 'VORTEX', 'OLYMPUS', 'CHRONOS', 'OBSIDIAN'];
    state.vigenereWord = keys[Math.floor(Math.random() * keys.length)];
  }
  runTransformation(true);
  showToast('Generated random key');
}

function onMessageInput() {
  playTactileFeedback(750, 0.009);
  runTransformation(false);
}

function swapResultToInput() {
  playTactileFeedback(820, 0.025);
  if (!state.lastRawResult) return;
  messageText.value = state.lastRawResult;
  state.action = state.action === 'encrypt' ? 'decrypt' : 'encrypt';
  runTransformation(true);
  showToast('Result swapped into input');
}

function clearAllFields() {
  playTactileFeedback(480, 0.02);
  messageText.value = '';
  state.lastRawResult = '';
  runTransformation(false);
  showToast('Cleared all fields');
}

function loadPresetSample() {
  playTactileFeedback(880, 0.025);
  const quotes = [
    "Order and chaos are two faces of the same mask",
    "The rites of Dionysus conceal truth behind sacred madness",
    "Veni vidi vici the quick onyx goblin jumps over the lazy dwarf"
  ];
  messageText.value = quotes[Math.floor(Math.random() * quotes.length)];
  runTransformation(true);
  showToast('Loaded sample text');
}

function copyResult() {
  playTactileFeedback(920, 0.02);
  const text = streamTokens.value.map(t => t.char).join('');
  if (!text || text.includes('Waiting for message')) return;
  navigator.clipboard.writeText(text.trim());
  copyBtnText.value = 'Copied!';
  setTimeout(() => (copyBtnText.value = 'Copy'), 1800);
  showToast('Copied to clipboard');
}

function downloadTextPayload() {
  playTactileFeedback(840, 0.02);
  const text = streamTokens.value.map(t => t.char).join('');
  if (!text || text.includes('Waiting for message')) return;

  const content = `--- DIONYSUS CRYPTOGRAPHIC STUDIO ---
ALGORITHM: ${state.method.toUpperCase()}
OPERATION: ${state.action.toUpperCase()}
FORMAT: ${state.spacingMode.toUpperCase()}
RESULT:
${text.trim()}
`;
  const blob = new Blob([content], { type: 'text/plain;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `dionysus_${state.method}_${state.action}.txt`;
  a.click();
  URL.revokeObjectURL(url);
  showToast('Saved file as .txt');
}

function toggleInspectorDrawer() {
  playTactileFeedback(680, 0.02);
  drawerOpen.value = !drawerOpen.value;
}

function getShiftPreview(s) {
  if (!messageText.value) return 'No text';
  const shifted = cipherCaesar(messageText.value.slice(0, 32), s, false);
  return formatWithSpaces(shifted);
}

function applyShift(s) {
  state.method = 'caesar';
  state.caesarShift = s;
  drawerOpen.value = false;
  runTransformation(true);
  showToast(`Shift set to +${s}`);
}

function openMethodsModal() {
  playTactileFeedback(680, 0.02);
  state.modalOpen = true;
}

function closeMethodsModal() {
  state.modalOpen = false;
  playTactileFeedback(420, 0.03);
}

function useMethodFromOverview(id) {
  state.method = id;
  state.activeTab = 'cipher';
  runTransformation(true);
}

function showToast(msg) {
  toastMsg.value = msg;
  toastVisible.value = true;
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => (toastVisible.value = false), 1800);
}

onMounted(() => {
  const minTimer = setTimeout(() => {
    showSplash.value = false;
  }, 1150);
  const guardTimer = setTimeout(() => {
    showSplash.value = false;
  }, 9000);

  runTransformation(true);
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape') {
      state.modalOpen = false;
      drawerOpen.value = false;
    }
  });

  return () => {
    clearTimeout(minTimer);
    clearTimeout(guardTimer);
  };
});

onBeforeUnmount(() => {
  clearInterval(scrambleInterval);
});
</script>

<style scoped>
/* Boot Splash Animation (#boot) */
#boot {
  position: fixed;
  inset: 0;
  background: #000000;
  z-index: 999999;
  display: flex;
  align-items: center;
  justify-content: center;
}

.boot-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.mark-wrapper {
  position: relative;
  width: 74px;
  height: 74px;
  margin-bottom: 20px;
}

.spin-ring {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  border: 2px solid transparent;
  border-top-color: #ffffff;
  border-right-color: #555555;
  animation: spinAction 1s linear infinite;
}

.d-symbol {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2.2rem;
  font-weight: 800;
  color: #ffffff;
}

.wordmark-track {
  display: flex;
  gap: 0.15rem;
}

.wordmark-track span {
  display: inline-block;
  font-size: 1.35rem;
  font-weight: 800;
  letter-spacing: 0.35rem;
  color: #ffffff;
  opacity: 0;
  animation: wordmarkEntrance 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

.boot-sub {
  font-size: 0.65rem;
  color: #777777;
  letter-spacing: 0.1em;
  margin-top: 14px;
}

@keyframes spinAction {
  100% { transform: rotate(360deg); }
}

@keyframes wordmarkEntrance {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.boot-fade-enter-active, .boot-fade-leave-active { transition: opacity 0.35s ease; }
.boot-fade-enter-from, .boot-fade-leave-to { opacity: 0; }

/* Page Layout Core */
.dionysus-page {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  background: #000000;
  color: #f5f5f5;
  overflow: hidden;
}

.dionysus-content {
  --background: #000000;
  --color: #f5f5f5;
  height: 100%;
  --overflow: hidden auto;
  scrollbar-width: none;
}

/* Kill all native white scrollbar leaks */
.dionysus-content::-webkit-scrollbar,
::-webkit-scrollbar {
  display: none !important;
  width: 0 !important;
  height: 0 !important;
  background: transparent !important;
}

.dionysus-shell {
  width: 100%;
  max-width: 52rem;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 0.85rem;
  padding-top: calc(env(safe-area-inset-top, 24px) + 0.65rem);
  padding-left: max(0.9rem, env(safe-area-inset-left));
  padding-right: max(0.9rem, env(safe-area-inset-right));
  padding-bottom: calc(5.2rem + env(safe-area-inset-bottom));
  box-sizing: border-box;
}

/* Header: 2-Row Stack sa Mobile para Walang Putol */
.app-header {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding-bottom: 0.65rem;
  border-bottom: 1px solid #1a1a1a;
}

.header-main-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.4rem;
  width: 100%;
}

.brand-cluster {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 0;
  flex-shrink: 0;
}

.brand-avatar {
  width: 1.85rem;
  height: 1.85rem;
  border-radius: 6px;
  background: #0d0d0d;
  border: 1px solid #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ffffff;
  font-weight: 800;
  font-size: 0.95rem;
  flex-shrink: 0;
}

.brand-titles {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 1rem;
  font-weight: 800;
  margin: 0;
  color: #f5f5f5;
  line-height: 1.1;
  letter-spacing: 0.02em;
  white-space: nowrap !important;
  word-break: keep-all !important;
  flex-shrink: 0;
}

@media (min-width: 480px) {
  .brand-name {
    font-size: 1.1rem;
  }
}

.header-sub-row {
  display: flex;
  align-items: center;
}

.brand-tagline {
  font-size: 0.65rem;
  color: #777777;
  margin: 0;
  line-height: 1.2;
  white-space: nowrap;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 0.2rem;
  flex-shrink: 0;
}

.header-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.2rem;
  min-height: 28px;
  padding: 0.2rem 0.4rem;
  border-radius: 5px;
  background: #080808;
  border: 1px solid #282828;
  color: #d4d4d4;
  font-size: 0.68rem;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  flex-shrink: 0;
}

/* Audio button text: hidden sa mobile para magkasya lahat kasama ang Sample, lalabas sa desktop */
.header-sound-text {
  display: none;
}
@media (min-width: 640px) {
  .header-sound-text {
    display: inline;
  }
}

.sound-btn {
  min-width: 26px;
  padding: 0.2rem 0.35rem;
}

.shifts-btn {
  white-space: nowrap;
  flex-shrink: 0;
}

.shifts-text {
  white-space: nowrap;
  display: inline;
}

.sample-pill {
  font-weight: 700;
  color: #f5f5f5;
  white-space: nowrap;
  flex-shrink: 0;
  padding: 0.2rem 0.5rem;
}

.header-btn:hover { background: #111111; border-color: #ffffff; color: #f5f5f5; }
.header-icon { width: 0.7rem; height: 0.7rem; color: #ffffff; flex-shrink: 0; }

/* Tabs */
.view-tabs {
  position: relative;
  display: flex;
  background: #070707;
  border: 1px solid #1a1a1a;
  border-radius: 8px;
  padding: 2px;
  isolation: isolate;
}

.view-tabs-pill {
  position: absolute;
  top: 2px;
  left: 2px;
  width: calc(50% - 2px);
  height: calc(100% - 4px);
  background: linear-gradient(180deg, #1c1c1c 0%, #101010 100%);
  border: 1px solid #3a3a3a;
  border-radius: 6px;
  transition: transform 0.28s cubic-bezier(0.16, 1, 0.3, 1);
  z-index: 1;
  pointer-events: none;
}

.view-tabs-pill.is-parked { opacity: 0.55; }

.view-tab {
  position: relative;
  z-index: 2;
  flex: 1;
  min-height: 32px;
  padding: 0.3rem;
  background: transparent;
  border: 0;
  color: #888888;
  font-family: 'Syne', sans-serif;
  font-size: 0.74rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  cursor: pointer;
}

.view-tab.active { color: #ffffff; }

/* Studio Grid Layout */
.studio-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 0.8rem;
}

@media (min-width: 1024px) {
  .studio-grid {
    grid-template-columns: 5fr 7fr;
  }
}

.studio-col {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.studio-panel {
  background: #070707;
  border: 1px solid #202020;
  border-radius: 11px;
  padding: 0.9rem 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.7rem;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #1a1a1a;
  padding-bottom: 0.5rem;
}

.panel-title {
  font-size: 0.78rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #d4d4d4;
  margin: 0;
}

.method-tag {
  font-size: 0.68rem;
  color: #ffffff;
  border: 1px solid #2a2a2a;
  padding: 2px 7px;
  border-radius: 5px;
  background: #111111;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.key-config-group {
  margin-top: 0.2rem;
}

.section-label {
  font-size: 0.68rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: #888888;
  line-height: 1.35;
}

.sub-label {
  font-size: 0.68rem;
  color: #888888;
  line-height: 1.35;
}

.row-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.45rem;
}

.key-hint {
  font-size: 0.68rem;
  color: #ffffff;
}

/* Strict SVG Size Constraints */
.control-icon,
.btn-inline-icon,
.card-arrow-icon,
.readmore-arrow-icon {
  width: 14px !important;
  height: 14px !important;
  max-width: 14px !important;
  max-height: 14px !important;
  flex-shrink: 0;
}

/* Form Controls */
.field-input {
  width: 100%;
  background: #050505;
  border: 1px solid #282828;
  border-radius: 7px;
  padding: 0.55rem 0.75rem;
  color: #ffffff;
  font-size: 0.82rem;
  outline: none;
  box-sizing: border-box;
}

.field-input:focus { border-color: #ffffff; }

.textarea-field {
  resize: vertical;
  min-height: 70px;
  line-height: 1.45;
}

.segmented-track {
  position: relative;
  display: flex;
  background: #000000;
  border: 1px solid #242424;
  border-radius: 7px;
  padding: 2px;
  isolation: isolate;
}

.pill-slider {
  position: absolute;
  top: 2px;
  left: 2px;
  bottom: 2px;
  width: calc(50% - 2px);
  background: #181818;
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 5px;
  transition: transform 0.22s ease;
  z-index: 1;
  pointer-events: none;
}

.segmented-btn {
  position: relative;
  z-index: 2;
  flex: 1;
  min-height: 30px;
  padding: 3px 8px;
  background: transparent;
  border: none;
  color: #888888;
  font-size: 0.74rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.segmented-btn.active { color: #ffffff; font-weight: 700; }

.method-picker-btn {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.6rem;
  width: 100%;
  min-height: 40px;
  padding: 0.45rem 0.75rem;
  background: #0a0a0a;
  border: 1px solid #282828;
  border-radius: 9px;
  cursor: pointer;
}

.method-picker-stack {
  display: flex;
  flex-direction: column;
  text-align: left;
}

.picker-prefix {
  font-size: 0.55rem;
  color: #666666;
  letter-spacing: 0.05em;
}

.method-picker-name {
  font-size: 0.82rem;
  font-weight: 700;
  color: #f5f5f5;
  line-height: 1.2;
}

.method-picker-action {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  padding: 0.25rem 0.5rem;
  background: #141414;
  border: 1px solid #282828;
  border-radius: 5px;
}

.action-text {
  font-size: 0.62rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #a3a3a3;
  letter-spacing: 0.04em;
}

.chevron-icon { width: 0.75rem; height: 0.75rem; color: #888888; }
.picker-family-desc { font-size: 0.68rem; color: #737373; margin: 0; line-height: 1.35; }

.no-key-note {
  font-size: 0.68rem;
  color: #888888;
  background: #050505;
  border: 1px solid #1a1a1a;
  border-radius: 7px;
  padding: 0.55rem 0.75rem;
  line-height: 1.4;
  margin: 0;
}

.affine-grid { display: flex; gap: 0.5rem; align-items: flex-end; }
.input-subgroup { flex: 1; display: flex; flex-direction: column; gap: 0.25rem; }
.select-wrapper { position: relative; display: flex; align-items: center; }
.select-styled { appearance: none; padding-right: 1.8rem; cursor: pointer; }
.select-chevron { position: absolute; right: 0.65rem; width: 0.8rem; height: 0.8rem; color: #888888; pointer-events: none; }

.key-input-row { display: flex; flex-wrap: wrap; gap: 0.45rem; align-items: center; }
.num-field { flex: 1; min-width: 75px; font-weight: 700; }
.text-field { flex: 1; font-weight: 700; }

.stepper-cluster {
  display: flex;
  gap: 0.25rem;
  flex-shrink: 0;
  width: 100%;
}
@media (min-width: 640px) { .stepper-cluster { width: auto; } }

.step-btn {
  flex: 1;
  min-height: 32px;
  padding: 0.35rem 0.55rem;
  background: #0a0a0a;
  border: 1px solid #2a2a2a;
  border-radius: 6px;
  color: #d4d4d4;
  font-size: 0.74rem;
  font-weight: 700;
  cursor: pointer;
}

.step-btn:hover { background: #141414; color: #ffffff; }
.highlight-btn { color: #ffffff; }
.slider-box { display: flex; align-items: center; padding-top: 0.2rem; }

input[type=range] {
  appearance: none;
  -webkit-appearance: none;
  width: 100%;
  height: 4px;
  background: #1a1a1a;
  border-radius: 2px;
  outline: none;
}

input[type=range]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: #ffffff;
  border: 2px solid #000000;
  cursor: pointer;
}

/* Canvas IO */
.output-header-row { flex-wrap: wrap; }
.output-label-box { display: flex; align-items: center; gap: 0.45rem; }
.format-badge {
  font-size: 0.62rem;
  padding: 1px 5px;
  border-radius: 4px;
  background: #111111;
  border: 1px solid #2a2a2a;
  color: #ffffff;
}

.output-actions-box { display: flex; align-items: center; gap: 0.35rem; }
.pill-btn {
  font-size: 0.68rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: #080808;
  border: 1px solid #2a2a2a;
  color: #a3a3a3;
  cursor: pointer;
}

.copy-action-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.74rem;
  font-weight: 600;
  padding: 3px 9px;
  border-radius: 6px;
  background: #0d0d0d;
  border: 1px solid #ffffff;
  color: #f5f5f5;
  cursor: pointer;
}

.card-surface {
  background: #0d0d0d;
  border: 1px solid #222222;
  border-radius: 10px;
}

.output-canvas {
  background: #040404;
  padding: 0.85rem;
  min-height: 85px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.canvas-top-telemetry { display: flex; justify-content: flex-end; margin-bottom: 0.25rem; }
.telemetry-tag { font-size: 0.62rem; color: #525252; letter-spacing: 0.05em; }
.telemetry-tag.is-active { color: #ffffff; animation: pulse-telemetry 0.5s infinite alternate; }

.canvas-text {
  font-size: 0.88rem;
  color: #fafafa;
  line-height: 1.6;
  word-break: break-all;
  user-select: text;
  min-height: 2.2rem;
}

.canvas-raw-preview {
  font-size: 0.65rem;
  color: #737373;
  border-top: 1px solid #141414;
  padding-top: 0.45rem;
  margin-top: 0.65rem;
  word-break: break-all;
  line-height: 1.35;
}

.stream-char { display: inline-block; }
.stream-char.is-scrambling { color: #555555; transform: translateY(-1px); }
.blinking-cursor {
  display: inline-block;
  width: 5px;
  height: 13px;
  background-color: #ffffff;
  margin-left: 3px;
  vertical-align: middle;
  animation: cursor-blink 0.8s infinite;
}

@keyframes cursor-blink { 0%, 100% { opacity: 1; } 50% { opacity: 0; } }
@keyframes pulse-telemetry { from { opacity: 0.4; } to { opacity: 1; } }
.spaced-char-stream { letter-spacing: 0.22em; word-spacing: 0.45em; }

/* Bottom Action Buttons */
.bottom-actions-row { display: flex; flex-wrap: wrap; gap: 0.5rem; }
.bottom-btn {
  flex: 1;
  min-height: 38px;
  min-width: 120px;
  border-radius: 10px;
  background: #080808;
  border: 1px solid #2a2a2a;
  color: #a3a3a3;
  font-size: 0.74rem;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  padding: 0.45rem 0.65rem;
  cursor: pointer;
  white-space: nowrap;
}

.bottom-btn:hover { background: #111111; border-color: #ffffff; color: #f5f5f5; }
.swap-action-btn { background: #0d0d0d; color: #f5f5f5; }
.clear-action-btn { flex: 0 0 auto; min-width: 60px; background: transparent; border-color: #222222; color: #888888; }
.rotator { transition: transform 0.35s cubic-bezier(0.16, 1, 0.3, 1); }
.swap-action-btn:hover .rotator { transform: rotate(180deg); }

/* Alignment View */
.alignment-panel { gap: 0.75rem; }
.entropy-badge { font-size: 0.7rem; color: #888888; }
.alignment-caption { font-size: 0.72rem; color: #777777; margin: 0; }
.active-cipher-name { color: #ffffff; font-weight: 600; }
.text-dim { color: #525252; }

.tape-viewport {
  background: #040404;
  border: 1px solid #1f1f1f;
  border-radius: 10px;
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
  overflow: hidden;
}

.tape-title-row { display: flex; align-items: center; gap: 0.4rem; }
.status-dot { width: 5px; height: 5px; border-radius: 50%; background: #ffffff; }
.tape-title-text { font-size: 0.62rem; font-weight: 700; text-transform: uppercase; color: #888888; }

.alignment-ribbon {
  display: flex;
  overflow-x: auto;
  gap: 4px;
  padding: 3px 0;
  scrollbar-width: none;
}
.alignment-ribbon::-webkit-scrollbar { display: none; }

.ribbon-cell {
  flex: 0 0 25px;
  height: 40px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  background: #060606;
  border: 1px solid #1a1a1a;
}

.ribbon-cell.shifted-highlight { background: #101010; border-color: rgba(255, 255, 255, 0.5); }
.cell-top { font-size: 8.5px; color: #888888; }
.cell-bot { font-size: 10.5px; font-weight: 700; color: #fafafa; }

/* Fixed Bottom CTAs */
.dionysus-cta {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 40;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 0.55rem 0.85rem calc(0.55rem + env(safe-area-inset-bottom));
  background: linear-gradient(to top, #000000 75%, transparent);
}

.dionysus-cta-inner { width: 100%; max-width: 52rem; margin: 0 auto; }
.dionysus-cta-btn {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  width: 100%;
  min-height: 42px;
  padding: 0.55rem 0.85rem;
  background: #0a0a0a;
  border: 1px solid #222222;
  border-radius: 12px;
  color: #f5f5f5;
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
}

.dionysus-cta-btn:hover { background: #121212; border-color: #444444; }
.cta-left-icon { width: 1rem; height: 1rem; color: #ffffff; flex-shrink: 0; }
.cta-btn-text { flex: 1; text-align: left; }
.cta-arrow-icon { width: 0.85rem; height: 0.85rem; color: #888888; flex-shrink: 0; }
.dionysus-cta-hint { font-size: 0.65rem; text-align: center; color: #737373; margin: 0; }

/* Methods Cards */
.methods-container { display: flex; flex-direction: column; gap: 0.85rem; }
.intro-panel { gap: 0.35rem; }
.compendium-sub { font-size: 0.74rem; color: #888888; line-height: 1.45; margin: 0; }

.method-card-grid { display: grid; grid-template-columns: 1fr; gap: 0.75rem; }
@media (min-width: 640px) { .method-card-grid { grid-template-columns: repeat(2, 1fr); } }

.method-card {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
  padding: 0.85rem 1rem;
  background: #070707;
  border: 1px solid #1c1c1c;
  border-radius: 14px;
}
.method-card.is-active { border-color: #444444; background: #0b0b0b; }

.method-head-row { display: flex; justify-content: space-between; align-items: flex-start; gap: 0.45rem; }
.method-title { font-size: 0.88rem; font-weight: 700; color: #f5f5f5; margin: 0; }
.method-meta { font-size: 0.62rem; text-transform: uppercase; color: #666666; margin: 0; }
.active-badge { font-size: 0.55rem; font-weight: 700; text-transform: uppercase; padding: 2px 6px; background: #ffffff; color: #000000; border-radius: 999px; }
.method-desc { font-size: 0.74rem; color: #a3a3a3; line-height: 1.4; margin: 0; }
.method-formula {
  padding: 0.4rem 0.55rem;
  background: #040404;
  border: 1px solid #1a1a1a;
  border-left: 2px solid #333333;
  border-radius: 6px;
  font-size: 0.68rem;
  color: #f5f5f5;
  white-space: nowrap;
  overflow-x: auto;
}

.method-card-example { font-size: 0.62rem; color: #888888; margin: 0; }
.method-card-notes { display: flex; flex-direction: column; gap: 0.3rem; margin: 0; }
.method-card-notes dt { font-size: 0.55rem; text-transform: uppercase; color: #525252; font-weight: 700; }
.method-card-notes dd { font-size: 0.68rem; color: #888888; margin: 0; line-height: 1.35; }

.method-card-use {
  display: flex;
  justify-content: space-between;
  align-items: center;
  min-height: 34px;
  padding: 0.45rem 0.65rem;
  background: #0d0d0d;
  border: 1px solid #2a2a2a;
  border-radius: 8px;
  font-size: 0.68rem;
  font-weight: 700;
  text-transform: uppercase;
  color: #f5f5f5;
  cursor: pointer;
  margin-top: 0.25rem;
}
.method-card-use:hover:not(:disabled) { background: #ffffff; color: #000000; }

/* Centered Dialog Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 60;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1.25rem;
  background: rgba(0, 0, 0, 0.82);
  backdrop-filter: blur(4px);
}

.modal-dialog-panel {
  width: 100%;
  max-width: 26rem;
  max-height: 82vh;
  background: #080808;
  border: 1px solid #2a2a2a;
  border-radius: 14px;
  box-shadow: 0 12px 36px rgba(0, 0, 0, 0.9);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 0.95rem;
  border-bottom: 1px solid #1c1c1c;
}

.modal-title { font-size: 0.88rem; font-weight: 700; color: #f5f5f5; margin: 0; }
.modal-sub { font-size: 0.68rem; color: #888888; margin: 0; line-height: 1.35; }

.modal-close {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1.75rem;
  height: 1.75rem;
  background: #111111;
  border: 1px solid #2a2a2a;
  border-radius: 6px;
  color: #a3a3a3;
  cursor: pointer;
}

.modal-list {
  flex: 1;
  overflow-y: auto;
  padding: 0.65rem 0.85rem;
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.modal-foot {
  padding: 0.65rem 0.85rem;
  border-top: 1px solid #1c1c1c;
  background: #050505;
}

.method-option {
  display: flex;
  align-items: flex-start;
  gap: 0.65rem;
  width: 100%;
  padding: 0.65rem 0.75rem;
  background: #050505;
  border: 1px solid #1a1a1a;
  border-radius: 8px;
  cursor: pointer;
}

.method-option:hover { background: #121212; border-color: #383838; }
.method-option.is-active { background: #141414; border-color: #ffffff; }

.method-option-tick {
  width: 1.35rem;
  height: 1.35rem;
  border-radius: 999px;
  background: #0d0d0d;
  border: 1px solid #2a2a2a;
  display: flex;
  align-items: center;
  justify-content: center;
  color: transparent;
  flex-shrink: 0;
}

.method-option.is-active .method-option-tick {
  background: #ffffff;
  color: #000000;
  border-color: #ffffff;
}

.method-option-name { font-size: 0.82rem; font-weight: 700; color: #f5f5f5; display: block; line-height: 1.25; }
.method-option-family { font-size: 0.62rem; text-transform: uppercase; color: #666666; display: block; }
.method-option-summary { font-size: 0.68rem; color: #888888; margin-top: 0.2rem; display: block; line-height: 1.35; }
.method-option-flag {
  font-size: 0.55rem;
  text-transform: uppercase;
  padding: 1px 5px;
  border-radius: 999px;
  background: #141414;
  border: 1px solid #2a2a2a;
  color: #a3a3a3;
  align-self: center;
}

.full-readmore-btn {
  width: 100%;
  min-height: 38px;
  padding: 0.45rem 0.85rem;
  background: #0a0a0a;
  border: 1px solid #2a2a2a;
  border-radius: 8px;
  color: #f5f5f5;
  font-size: 0.75rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  cursor: pointer;
}

/* All Shifts Drawer */
.drawer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.88);
  z-index: 50;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease;
}

.drawer-overlay.active { opacity: 1; pointer-events: auto; }

.drawer-panel {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  width: 100%;
  max-width: 380px;
  background: #080808;
  border-left: 1px solid #262626;
  transform: translateX(100%);
  transition: transform 0.25s cubic-bezier(0.16, 1, 0.3, 1);
  display: flex;
  flex-direction: column;
  padding-top: env(safe-area-inset-top, 16px);
}

.drawer-overlay.active .drawer-panel { transform: translateX(0); }

.drawer-head {
  padding: 0.75rem 0.85rem;
  border-bottom: 1px solid #1f1f1f;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.drawer-close-btn {
  font-size: 0.72rem;
  padding: 3px 8px;
  background: #0d0d0d;
  border: 1px solid #2a2a2a;
  color: #a3a3a3;
  border-radius: 5px;
  cursor: pointer;
}

.drawer-body {
  flex: 1;
  overflow-y: auto;
  padding: 0.85rem;
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.brute-card {
  padding: 0.55rem 0.65rem;
  border-radius: 6px;
  background: #070707;
  border: 1px solid #1f1f1f;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.brute-card:hover { border-color: #ffffff; }
.brute-text { font-size: 0.68rem; color: #fafafa; word-break: break-all; line-height: 1.35; }

/* TOP CENTERED FLOATING TOAST */
.toast-pill {
  position: fixed;
  left: 50% !important;
  right: auto !important;
  top: calc(env(safe-area-inset-top, 24px) + 0.65rem) !important;
  bottom: auto !important;
  transform: translate(-50%, -15px) !important;
  opacity: 0;
  pointer-events: none;
  background: #0d0d0d;
  border: 1px solid #ffffff;
  color: #ffffff;
  height: 28px;
  padding: 0 0.75rem;
  border-radius: 999px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.35rem;
  font-size: 0.7rem;
  font-weight: 600;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.95);
  z-index: 10000;
  width: max-content;
  max-width: min(85vw, 300px);
  white-space: nowrap;
  transition: opacity 0.2s cubic-bezier(0.16, 1, 0.3, 1), transform 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

.toast-pill.show {
  transform: translate(-50%, 0) !important;
  opacity: 1;
}

.toast-check-icon {
  width: 12px !important;
  height: 12px !important;
  max-width: 12px !important;
  max-height: 12px !important;
  min-width: 12px !important;
  min-height: 12px !important;
  flex-shrink: 0;
  color: #ffffff;
}

.toast-content {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Animations & Transitions */
.tab-fade-enter-active, .tab-fade-leave-active {
  transition: opacity 0.18s cubic-bezier(0.16, 1, 0.3, 1), transform 0.18s cubic-bezier(0.16, 1, 0.3, 1);
}
.tab-fade-enter-from { opacity: 0; transform: translateY(6px); }
.tab-fade-leave-to { opacity: 0; transform: translateY(-6px); }

.dialog-pop-enter-active, .dialog-pop-leave-active { transition: opacity 0.18s ease; }
.dialog-pop-enter-active .modal-dialog-panel, .dialog-pop-leave-active .modal-dialog-panel {
  transition: transform 0.24s cubic-bezier(0.16, 1, 0.3, 1), opacity 0.18s ease;
}
.dialog-pop-enter-from, .dialog-pop-leave-to { opacity: 0; }
.dialog-pop-enter-from .modal-dialog-panel, .dialog-pop-leave-to .modal-dialog-panel {
  transform: scale(0.92);
  opacity: 0;
}

.btn-tactile { transition: transform 0.1s ease; }
.btn-tactile:active { transform: scale(0.97); }

/* Typography Utility */
.font-brand { font-family: 'Syne', sans-serif; }
.font-mono-code { font-family: 'Share Tech Mono', monospace; }
.font-tech { font-family: 'Chakra Petch', sans-serif; }
.rotate-180 { transform: rotate(180deg); }
.truncate { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.text-muted { color: #525252; }
</style>
/\*\*  
 \* PayloadGuard Pro \- Studio Application Orchestrator  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  // Elements  
  const presetSelect \= document.getElementById('presetSelect');  
  const schemaTextarea \= document.getElementById('schemaTextarea');  
  const payloadTextarea \= document.getElementById('payloadTextarea');  
  const validationBanner \= document.getElementById('validationBanner');  
  const validationStatusText \= document.getElementById('validationStatusText');  
  const errorsList \= document.getElementById('errorsList');

  // Tabs  
  const tabButtons \= document.querySelectorAll('.tab-btn');  
  const tabContents \= document.querySelectorAll('.tab-content');

  // Code Tab Elements  
  const fwButtons \= document.querySelectorAll('.fw-tab-btn');  
  const generatedCodePre \= document.getElementById('generatedCodePre');  
  const copyCodeBtn \= document.getElementById('copyCodeBtn');

  // Fuzz Matrix Tab  
  const fuzzVectorsCount \= document.getElementById('fuzzVectorsCount');  
  const fuzzList \= document.getElementById('fuzzList');  
  const exportFuzzJsonBtn \= document.getElementById('exportFuzzJsonBtn');

  // Automated Tests Tab  
  const testFwButtons \= document.querySelectorAll('.test-fw-btn');  
  const generatedTestsPre \= document.getElementById('generatedTestsPre');  
  const copyTestsBtn \= document.getElementById('copyTestsBtn');

  // State  
  let activeFw \= 'zod';  
  let activeTestFw \= 'jest';  
  let currentSchema \= {};  
  let currentPayload \= {};  
  let currentFuzzVectors \= \[\];

  const PRESETS \= {};

  async function initPresets() {  
    try {  
      const p1 \= await fetch('presets/ecommerce-order-checkout.json').then(r \=\> r.json());  
      PRESETS\['ecommerce'\] \= p1;  
      const p2 \= await fetch('presets/user-onboarding-auth.json').then(r \=\> r.json());  
      PRESETS\['auth'\] \= p2;  
      const p3 \= await fetch('presets/iot-telemetry-event.json').then(r \=\> r.json());  
      PRESETS\['iot'\] \= p3;  
    } catch (e) {  
      console.warn('Presets loaded from local fallback');  
    }  
  }

  function evaluate() {  
    let schemaParsed \= null;  
    let payloadParsed \= null;

    try {  
      schemaParsed \= JSON.parse(schemaTextarea.value);  
    } catch (e) {  
      renderJsonError('Schema', e.message);  
      return;  
    }

    try {  
      payloadParsed \= JSON.parse(payloadTextarea.value);  
    } catch (e) {  
      renderJsonError('Payload', e.message);  
      return;  
    }

    currentSchema \= schemaParsed;  
    currentPayload \= payloadParsed;

    // Run Validation  
    const valResult \= ValidatorEngine.validate(currentSchema, currentPayload);  
    renderValidationStatus(valResult);

    // Synthesize Code  
    renderCode();

    // Synthesize Fuzz Vectors  
    currentFuzzVectors \= FuzzGenerator.generateFuzzMatrix(currentSchema, currentPayload);  
    renderFuzzMatrix();

    // Synthesize Unit Tests  
    renderTests();  
  }

  function renderValidationStatus(valResult) {  
    if (valResult.valid) {  
      validationBanner.className \= 'status-banner banner-valid';  
      validationStatusText.innerHTML \= '✅ \<strong\>Payload Valid\</strong\> — Conforms strictly to JSON Schema constraints.';  
      errorsList.innerHTML \= '';  
      errorsList.style.display \= 'none';  
    } else {  
      validationBanner.className \= 'status-banner banner-invalid';  
      validationStatusText.innerHTML \= \`❌ \<strong\>Validation Failed\</strong\> — \${valResult.errors.length} constraint violation(s) detected.\`;  
        
      let html \= '';  
      valResult.errors.forEach(err \=\> {  
        html \+= \`  
          \<div class="error-chip"\>  
            \<strong\>\${escapeHtml(err.path)}\</strong\>: \${escapeHtml(err.message)}  
          \</div\>  
        \`;  
      });  
      errorsList.innerHTML \= html;  
      errorsList.style.display \= 'flex';  
    }  
  }

  function renderJsonError(source, msg) {  
    validationBanner.className \= 'status-banner banner-invalid';  
    validationStatusText.innerHTML \= \`⚠️ \<strong\>JSON Syntax Error\</strong\> in \${source}: \${escapeHtml(msg)}\`;  
    errorsList.innerHTML \= '';  
    errorsList.style.display \= 'none';  
  }

  function renderCode() {  
    const title \= currentSchema.title || 'Payload';  
    let code \= '';  
    if (activeFw \=== 'zod') {  
      code \= SchemaSynthesizer.toZod(currentSchema, title);  
    } else if (activeFw \=== 'pydantic') {  
      code \= SchemaSynthesizer.toPydantic(currentSchema, title);  
    } else if (activeFw \=== 'ts') {  
      code \= SchemaSynthesizer.toTypeScript(currentSchema, title);  
    } else if (activeFw \=== 'golang') {  
      code \= SchemaSynthesizer.toGolang(currentSchema, title);  
    }  
    generatedCodePre.textContent \= code;  
  }

  function renderFuzzMatrix() {  
    fuzzVectorsCount.textContent \= \`\${currentFuzzVectors.length} Vectors Generated\`;  
    let html \= '';

    currentFuzzVectors.forEach(v \=\> {  
      const isReject \= v.expectedOutcome \=== 'REJECT\_HTTP\_422';  
      html \+= \`  
        \<div class="fuzz-card"\>  
          \<div class="fuzz-header"\>  
            \<span class="fuzz-title"\>\${escapeHtml(v.id)}: \${escapeHtml(v.category)}\</span\>  
            \<span class="severity-pill \${isReject ? 'pill-REJECT' : 'pill-SEC'}"\>\${v.expectedOutcome}\</span\>  
          \</div\>  
          \<div style="font-size: 0.8rem; color: var(--text-secondary);"\>\${escapeHtml(v.description)}\</div\>  
          \<details style="margin-top: 0.35rem;"\>  
            \<summary style="font-size: 0.75rem; color: var(--color-brand); cursor: pointer;"\>View Payload\</summary\>  
            \<pre class="code-preview" style="max-height: 120px; font-size: 0.72rem; margin-top: 0.25rem;"\>\${escapeHtml(JSON.stringify(v.payload, null, 2))}\</pre\>  
          \</details\>  
        \</div\>  
      \`;  
    });  
    fuzzList.innerHTML \= html;  
  }

  function renderTests() {  
    const title \= currentSchema.title || 'Payload';  
    let code \= '';  
    if (activeTestFw \=== 'jest') {  
      code \= FuzzGenerator.toJestTestSuite(currentFuzzVectors, title);  
    } else if (activeTestFw \=== 'pytest') {  
      code \= FuzzGenerator.toPyTestSuite(currentFuzzVectors, title);  
    }  
    generatedTestsPre.textContent \= code;  
  }

  function escapeHtml(str) {  
    if (\!str) return '';  
    return String(str)  
      .replace(/&/g, '\&amp;')  
      .replace(/\</g, '\&lt;')  
      .replace(/\>/g, '\&gt;')  
      .replace(/"/g, '\&quot;')  
      .replace(/'/g, '&\#039;');  
  }

  // Event Listeners  
  schemaTextarea.addEventListener('input', () \=\> evaluate());  
  payloadTextarea.addEventListener('input', () \=\> evaluate());

  presetSelect.addEventListener('change', (e) \=\> {  
    const key \= e.target.value;  
    if (PRESETS\[key\]) {  
      schemaTextarea.value \= JSON.stringify(PRESETS\[key\].schema, null, 2);  
      payloadTextarea.value \= JSON.stringify(PRESETS\[key\].sample, null, 2);  
      evaluate();  
    }  
  });

  tabButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabButtons.forEach(b \=\> b.classList.remove('active'));  
      tabContents.forEach(c \=\> c.classList.remove('active'));  
      btn.classList.add('active');  
      const target \= btn.getAttribute('data-tab');  
      document.getElementById(target).classList.add('active');  
    });  
  });

  fwButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      fwButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeFw \= btn.getAttribute('data-fw');  
      renderCode();  
    });  
  });

  testFwButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      testFwButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeTestFw \= btn.getAttribute('data-test-fw');  
      renderTests();  
    });  
  });

  copyCodeBtn.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(generatedCodePre.textContent);  
    copyCodeBtn.textContent \= 'Copied\!';  
    setTimeout(() \=\> copyCodeBtn.textContent \= 'Copy Code', 1500);  
  });

  copyTestsBtn.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(generatedTestsPre.textContent);  
    copyTestsBtn.textContent \= 'Copied\!';  
    setTimeout(() \=\> copyTestsBtn.textContent \= 'Copy Test File', 1500);  
  });

  exportFuzzJsonBtn.addEventListener('click', () \=\> {  
    const data \= {  
      product: 'PayloadGuard Pro (v1.0.0)',  
      schemaTitle: currentSchema.title || 'Payload',  
      generatedAt: new Date().toISOString(),  
      vectorsCount: currentFuzzVectors.length,  
      vectors: currentFuzzVectors  
    };  
    const blob \= new Blob(\[JSON.stringify(data, null, 2)\], { type: 'application/json' });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= \`fuzz-vectors-\${currentSchema.title || 'payload'}.json\`;  
    a.click();  
    URL.revokeObjectURL(url);  
  });

  // Initialization  
  initPresets().then(() \=\> {  
    if (PRESETS\['ecommerce'\]) {  
      schemaTextarea.value \= JSON.stringify(PRESETS\['ecommerce'\].schema, null, 2);  
      payloadTextarea.value \= JSON.stringify(PRESETS\['ecommerce'\].sample, null, 2);  
    }  
    evaluate();  
  });  
});  

/\*\*  
 \* PayloadGuard Pro \- Automated Fuzz-Test Vector Matrix Generator  
 \* Synthesizes boundary, edge-case, and adversarial payloads for API hardening.  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.FuzzGenerator \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function() {

  function generateFuzzMatrix(schema, samplePayload \= {}) {  
    const vectors \= \[\];  
    const base \= JSON.parse(JSON.stringify(samplePayload));

    // Vector 1: Missing Required Fields  
    const reqs \= schema.required || \[\];  
    for (const reqKey of reqs) {  
      const payloadMissing \= JSON.parse(JSON.stringify(base));  
      delete payloadMissing\[reqKey\];  
      vectors.push({  
        id: \`FUZZ\_REQ\_OMIT\_\${reqKey.toUpperCase()}\`,  
        category: 'Missing Required Field',  
        description: \`Omit required property "\${reqKey}"\`,  
        expectedOutcome: 'REJECT\_HTTP\_422',  
        payload: payloadMissing  
      });  
    }

    // Vector 2: Type Coercion Attacks  
    const props \= schema.properties || {};  
    for (const \[k, p\] of Object.entries(props)) {  
      if (p.type \=== 'number' || p.type \=== 'integer') {  
        const payloadStr \= JSON.parse(JSON.stringify(base));  
        payloadStr\[k\] \= "not\_a\_number\_string";  
        vectors.push({  
          id: \`FUZZ\_TYPE\_COERCE\_NUM\_\${k.toUpperCase()}\`,  
          category: 'Type Coercion Injection',  
          description: \`Inject string "not\_a\_number\_string" into numeric property "\${k}"\`,  
          expectedOutcome: 'REJECT\_HTTP\_422',  
          payload: payloadStr  
        });  
      } else if (p.type \=== 'string') {  
        const payloadBool \= JSON.parse(JSON.stringify(base));  
        payloadBool\[k\] \= true;  
        vectors.push({  
          id: \`FUZZ\_TYPE\_COERCE\_STR\_\${k.toUpperCase()}\`,  
          category: 'Type Coercion Injection',  
          description: \`Inject boolean true into string property "\${k}"\`,  
          expectedOutcome: 'REJECT\_HTTP\_422',  
          payload: payloadBool  
        });  
      } else if (p.type \=== 'array') {  
        const payloadObj \= JSON.parse(JSON.stringify(base));  
        payloadObj\[k\] \= { malformed: "not\_an\_array" };  
        vectors.push({  
          id: \`FUZZ\_TYPE\_COERCE\_ARR\_\${k.toUpperCase()}\`,  
          category: 'Type Coercion Injection',  
          description: \`Inject object into array property "\${k}"\`,  
          expectedOutcome: 'REJECT\_HTTP\_422',  
          payload: payloadObj  
        });  
      }  
    }

    // Vector 3: Numeric Boundaries & Negative Extremes  
    for (const \[k, p\] of Object.entries(props)) {  
      if (p.type \=== 'number' || p.type \=== 'integer') {  
        if (p.minimum \!== undefined && p.minimum \>= 0\) {  
          const payloadNeg \= JSON.parse(JSON.stringify(base));  
          payloadNeg\[k\] \= \-1;  
          vectors.push({  
            id: \`FUZZ\_NUM\_NEGATIVE\_\${k.toUpperCase()}\`,  
            category: 'Boundary & Range Violation',  
            description: \`Inject negative integer \-1 into property "\${k}" (minimum: \${p.minimum})\`,  
            expectedOutcome: 'REJECT\_HTTP\_422',  
            payload: payloadNeg  
          });  
        }  
        // Integer overflow  
        const payloadOverflow \= JSON.parse(JSON.stringify(base));  
        payloadOverflow\[k\] \= 9007199254740991; // Number.MAX\_SAFE\_INTEGER  
        vectors.push({  
          id: \`FUZZ\_NUM\_OVERFLOW\_\${k.toUpperCase()}\`,  
          category: 'Boundary & Range Violation',  
          description: \`Inject MAX\_SAFE\_INTEGER (9007199254740991) into property "\${k}"\`,  
          expectedOutcome: 'BOUNDARY\_CHECK',  
          payload: payloadOverflow  
        });  
      }  
    }

    // Vector 4: String Security Injections (SQLi, XSS, Null Bytes, Buffer Overflow)  
    for (const \[k, p\] of Object.entries(props)) {  
      if (p.type \=== 'string') {  
        // Empty string  
        const payloadEmpty \= JSON.parse(JSON.stringify(base));  
        payloadEmpty\[k\] \= "";  
        vectors.push({  
          id: \`FUZZ\_STR\_EMPTY\_\${k.toUpperCase()}\`,  
          category: 'String Boundary',  
          description: \`Inject empty string into property "\${k}"\`,  
          expectedOutcome: p.minLength ? 'REJECT\_HTTP\_422' : 'EVALUATE',  
          payload: payloadEmpty  
        });

        // SQL Injection  
        const payloadSql \= JSON.parse(JSON.stringify(base));  
        payloadSql\[k\] \= "' OR '1'='1' \-- /\*";  
        vectors.push({  
          id: \`FUZZ\_SEC\_SQLI\_\${k.toUpperCase()}\`,  
          category: 'Security Injection',  
          description: \`Inject SQL syntax pattern into property "\${k}"\`,  
          expectedOutcome: 'SANITIZED\_OR\_ACCEPTED',  
          payload: payloadSql  
        });

        // XSS Payload  
        const payloadXss \= JSON.parse(JSON.stringify(base));  
        payloadXss\[k\] \= "\<script\>alert('pwned')\</script\>";  
        vectors.push({  
          id: \`FUZZ\_SEC\_XSS\_\${k.toUpperCase()}\`,  
          category: 'Security Injection',  
          description: \`Inject HTML/XSS script tag into property "\${k}"\`,  
          expectedOutcome: 'SANITIZED\_OR\_ACCEPTED',  
          payload: payloadXss  
        });

        // Null-byte injection  
        const payloadNull \= JSON.parse(JSON.stringify(base));  
        payloadNull\[k\] \= "admin\\u0000bypass";  
        vectors.push({  
          id: \`FUZZ\_SEC\_NULL\_BYTE\_\${k.toUpperCase()}\`,  
          category: 'Security Injection',  
          description: \`Inject null-byte \\\\u0000 into property "\${k}"\`,  
          expectedOutcome: 'REJECT\_OR\_STRIP',  
          payload: payloadNull  
        });  
        break; // Only test security vectors on the first string to avoid bloating  
      }  
    }

    // Vector 5: Array Edge Cases (Empty array)  
    for (const \[k, p\] of Object.entries(props)) {  
      if (p.type \=== 'array' && p.minItems) {  
        const payloadEmptyArr \= JSON.parse(JSON.stringify(base));  
        payloadEmptyArr\[k\] \= \[\];  
        vectors.push({  
          id: \`FUZZ\_ARR\_EMPTY\_\${k.toUpperCase()}\`,  
          category: 'Array Boundary',  
          description: \`Inject empty array \[\] into property "\${k}" (minItems: \${p.minItems})\`,  
          expectedOutcome: 'REJECT\_HTTP\_422',  
          payload: payloadEmptyArr  
        });  
      }  
    }

    return vectors;  
  }

  // Generate Jest / Vitest Unit Test File  
  function toJestTestSuite(vectors, title \= 'Payload') {  
    let out \= \`// \==========================================  
// Jest / Vitest Automated Fuzz Test Suite  
// Generated by PayloadGuard Pro v1.0.0  
// \==========================================  
import { describe, it, expect } from 'vitest';  
import { \${title}Schema } from './schemas';

describe('\${title} Validation & Fuzz Test Suite', () \=\> {\\n\`;

    vectors.forEach(v \=\> {  
      const isReject \= v.expectedOutcome \=== 'REJECT\_HTTP\_422';  
      out \+= \`  it('\${v.id}: \${v.description}', () \=\> {\\n\`;  
      out \+= \`    const payload \= \${JSON.stringify(v.payload, null, 2).replace(/\\n/g, '\\n    ')};\\n\`;  
      if (isReject) {  
        out \+= \`    const result \= \${title}Schema.safeParse(payload);\\n\`;  
        out \+= \`    expect(result.success).toBe(false);\\n\`;  
      } else {  
        out \+= \`    // Verify schema sanitizes or safely processes boundary input\\n\`;  
        out \+= \`    expect(() \=\> \${title}Schema.parse(payload)).not.toThrow();\\n\`;  
      }  
      out \+= \`  });\\n\\n\`;  
    });

    out \+= \`});\\n\`;  
    return out;  
  }

  // Generate PyTest Test File  
  function toPyTestSuite(vectors, title \= 'Payload') {  
    let out \= \`\# \==========================================  
\# PyTest Automated Fuzz Test Suite  
\# Generated by PayloadGuard Pro v1.0.0  
\# \==========================================  
import pytest  
from pydantic import ValidationError  
from models import \${title}

\`;  
    vectors.forEach(v \=\> {  
      const isReject \= v.expectedOutcome \=== 'REJECT\_HTTP\_422';  
      const funcName \= v.id.toLowerCase();  
      out \+= \`def test\_\${funcName}():\\n\`;  
      out \+= \`    \\"\\"\\"\${v.description}\\"\\"\\"\\n\`;  
      out \+= \`    payload \= \${JSON.stringify(v.payload, null, 4).replace(/\\n/g, '\\n    ')}\\n\`;  
      if (isReject) {  
        out \+= \`    with pytest.raises(ValidationError):\\n\`;  
        out \+= \`        \${title}(\*\*payload)\\n\\n\`;  
      } else {  
        out \+= \`    \# Must parse without unhandled exception\\n\`;  
        out \+= \`    obj \= \${title}(\*\*payload)\\n\`;  
        out \+= \`    assert obj is not None\\n\\n\`;  
      }  
    });

    return out;  
  }

  return {  
    generateFuzzMatrix,  
    toJestTestSuite,  
    toPyTestSuite  
  };  
}));  

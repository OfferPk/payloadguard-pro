/\*\*  
 \* PayloadGuard Pro \- Multi-Model Schema Synthesizer  
 \* Generates Zod v3, Pydantic v2, TypeScript, and Golang types from JSON Schema.  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SchemaSynthesizer \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function() {

  function toPascalCase(str) {  
    return str  
      .replace(/\[-\_ \]+(\\w)/g, (\_, c) \=\> c.toUpperCase())  
      .replace(/^\\w/, c \=\> c.toUpperCase());  
  }

  // 1\. Zod v3 Generator  
  function toZod(schema, title \= 'Payload') {  
    const mainTitle \= toPascalCase(schema.title || title);  
    let out \= \`import { z } from 'zod';\\n\\n\`;  
    const nestedDefinitions \= \[\];

    function parseProp(prop, isRequired \= true) {  
      let zType \= 'z.unknown()';

      if (prop.enum) {  
        zType \= \`z.enum(\[\${prop.enum.map(e \=\> \`'\${e}'\`).join(', ')}\])\`;  
      } else if (prop.type \=== 'string') {  
        zType \= 'z.string()';  
        if (prop.format \=== 'email') zType \+= '.email()';  
        if (prop.minLength \!== undefined) zType \+= \`.min(\${prop.minLength})\`;  
        if (prop.maxLength \!== undefined) zType \+= \`.max(\${prop.maxLength})\`;  
        if (prop.pattern) zType \+= \`.regex(/\${prop.pattern}/)\`;  
      } else if (prop.type \=== 'number' || prop.type \=== 'integer') {  
        zType \= prop.type \=== 'integer' ? 'z.number().int()' : 'z.number()';  
        if (prop.minimum \!== undefined) zType \+= \`.min(\${prop.minimum})\`;  
        if (prop.maximum \!== undefined) zType \+= \`.max(\${prop.maximum})\`;  
      } else if (prop.type \=== 'boolean') {  
        zType \= 'z.boolean()';  
      } else if (prop.type \=== 'array') {  
        const itemType \= prop.items ? parseProp(prop.items, true) : 'z.unknown()';  
        zType \= \`z.array(\${itemType})\`;  
        if (prop.minItems \!== undefined) zType \+= \`.min(\${prop.minItems})\`;  
        if (prop.maxItems \!== undefined) zType \+= \`.max(\${prop.maxItems})\`;  
      } else if (prop.type \=== 'object') {  
        const reqs \= prop.required || \[\];  
        const props \= prop.properties || {};  
        const lines \= Object.entries(props).map((\[k, v\]) \=\> {  
          const req \= reqs.includes(k);  
          return \`  \${k}: \${parseProp(v, req)}\`;  
        });  
        zType \= \`z.object({\\n\${lines.join(',\\n')}\\n})\`;  
      }

      if (\!isRequired) {  
        zType \+= '.optional()';  
      }  
      return zType;  
    }

    const reqs \= schema.required || \[\];  
    const props \= schema.properties || {};  
    const lines \= Object.entries(props).map((\[k, v\]) \=\> {  
      const isReq \= reqs.includes(k);  
      return \`  \${k}: \${parseProp(v, isReq)}\`;  
    });

    out \+= \`export const \${mainTitle}Schema \= z.object({\\n\${lines.join(',\\n')}\\n});\\n\\n\`;  
    out \+= \`export type \${mainTitle} \= z.infer\<typeof \${mainTitle}Schema\>;\\n\`;  
    return out;  
  }

  // 2\. Pydantic v2 Generator  
  function toPydantic(schema, title \= 'Payload') {  
    const mainTitle \= toPascalCase(schema.title || title);  
    let out \= \`from typing import List, Optional, Literal\\nfrom pydantic import BaseModel, Field, EmailStr, ConfigDict\\n\\n\`;  
    const subClasses \= \[\];

    function parseProp(propKey, prop, isRequired \= true) {  
      let pyType \= 'object';  
      let fieldArgs \= \[\];

      if (prop.enum) {  
        pyType \= \`Literal\[\${prop.enum.map(e \=\> \`"\${e}"\`).join(', ')}\]\`;  
      } else if (prop.type \=== 'string') {  
        pyType \= prop.format \=== 'email' ? 'EmailStr' : 'str';  
        if (prop.minLength \!== undefined) fieldArgs.push(\`min\_length=\${prop.minLength}\`);  
        if (prop.maxLength \!== undefined) fieldArgs.push(\`max\_length=\${prop.maxLength}\`);  
        if (prop.pattern) fieldArgs.push(\`pattern=r"\${prop.pattern}"\`);  
      } else if (prop.type \=== 'integer') {  
        pyType \= 'int';  
        if (prop.minimum \!== undefined) fieldArgs.push(\`ge=\${prop.minimum}\`);  
        if (prop.maximum \!== undefined) fieldArgs.push(\`le=\${prop.maximum}\`);  
      } else if (prop.type \=== 'number') {  
        pyType \= 'float';  
        if (prop.minimum \!== undefined) fieldArgs.push(\`ge=\${prop.minimum}\`);  
        if (prop.maximum \!== undefined) fieldArgs.push(\`le=\${prop.maximum}\`);  
      } else if (prop.type \=== 'boolean') {  
        pyType \= 'bool';  
      } else if (prop.type \=== 'array') {  
        if (prop.items && prop.items.type \=== 'object') {  
          const subName \= \`\${toPascalCase(propKey)}Item\`;  
          generateSubClass(subName, prop.items);  
          pyType \= \`List\[\${subName}\]\`;  
        } else {  
          pyType \= 'List\[str\]';  
        }  
      } else if (prop.type \=== 'object') {  
        const subName \= toPascalCase(propKey);  
        generateSubClass(subName, prop);  
        pyType \= subName;  
      }

      if (\!isRequired) {  
        pyType \= \`Optional\[\${pyType}\]\`;  
        fieldArgs.push('default=None');  
      } else {  
        fieldArgs.unshift('...');  
      }

      const fieldStr \= fieldArgs.length \> 0 ? \`Field(\${fieldArgs.join(', ')})\` : '';  
      return { pyType, fieldStr };  
    }

    function generateSubClass(className, subSchema) {  
      let s \= \`class \${className}(BaseModel):\\n\`;  
      const sReqs \= subSchema.required || \[\];  
      const sProps \= subSchema.properties || {};  
      for (const \[k, v\] of Object.entries(sProps)) {  
        const isR \= sReqs.includes(k);  
        const { pyType, fieldStr } \= parseProp(k, v, isR);  
        s \+= \`    \${k}: \${pyType} \= \${fieldStr || 'Field(...)'}\\n\`;  
      }  
      subClasses.push(s);  
    }

    // Process nested objects first  
    const reqs \= schema.required || \[\];  
    const props \= schema.properties || {};  
    const mainFields \= \[\];

    for (const \[k, v\] of Object.entries(props)) {  
      const isReq \= reqs.includes(k);  
      const { pyType, fieldStr } \= parseProp(k, v, isReq);  
      mainFields.push(\`    \${k}: \${pyType} \= \${fieldStr || 'Field(...)'}\`);  
    }

    if (subClasses.length \> 0\) {  
      out \+= subClasses.join('\\n\\n') \+ '\\n\\n';  
    }

    out \+= \`class \${mainTitle}(BaseModel):\\n\`;  
    out \+= \`    model\_config \= ConfigDict(extra='forbid', str\_strip\_whitespace=True)\\n\\n\`;  
    out \+= mainFields.join('\\n') \+ '\\n';  
    return out;  
  }

  // 3\. TypeScript Interface Generator  
  function toTypeScript(schema, title \= 'Payload') {  
    const mainTitle \= toPascalCase(schema.title || title);  
    let out \= '';  
    const subTypes \= \[\];

    function parseProp(propKey, prop) {  
      if (prop.enum) {  
        return prop.enum.map(e \=\> \`'\${e}'\`).join(' | ');  
      }  
      if (prop.type \=== 'string') return 'string';  
      if (prop.type \=== 'number' || prop.type \=== 'integer') return 'number';  
      if (prop.type \=== 'boolean') return 'boolean';  
      if (prop.type \=== 'array') {  
        if (prop.items && prop.items.type \=== 'object') {  
          const subName \= \`\${toPascalCase(propKey)}Item\`;  
          subTypes.push(generateInterface(subName, prop.items));  
          return \`\${subName}\[\]\`;  
        }  
        return 'string\[\]';  
      }  
      if (prop.type \=== 'object') {  
        const subName \= toPascalCase(propKey);  
        subTypes.push(generateInterface(subName, prop));  
        return subName;  
      }  
      return 'any';  
    }

    function generateInterface(name, s) {  
      let code \= \`export interface \${name} {\\n\`;  
      const sReqs \= s.required || \[\];  
      const sProps \= s.properties || {};  
      for (const \[k, v\] of Object.entries(sProps)) {  
        const opt \= sReqs.includes(k) ? '' : '?';  
        code \+= \`  \${k}\${opt}: \${parseProp(k, v)};\\n\`;  
      }  
      code \+= \`}\`;  
      return code;  
    }

    const reqs \= schema.required || \[\];  
    const props \= schema.properties || {};  
    let main \= \`export interface \${mainTitle} {\\n\`;  
    for (const \[k, v\] of Object.entries(props)) {  
      const opt \= reqs.includes(k) ? '' : '?';  
      main \+= \`  \${k}\${opt}: \${parseProp(k, v)};\\n\`;  
    }  
    main \+= \`}\\n\`;

    if (subTypes.length \> 0\) {  
      out \+= subTypes.join('\\n\\n') \+ '\\n\\n';  
    }  
    out \+= main;  
    return out;  
  }

  // 4\. Golang Struct Generator  
  function toGolang(schema, title \= 'Payload') {  
    const mainTitle \= toPascalCase(schema.title || title);  
    let out \= 'package models\\n\\n';  
    const subStructs \= \[\];

    function parseProp(propKey, prop) {  
      if (prop.type \=== 'string') return 'string';  
      if (prop.type \=== 'integer') return 'int64';  
      if (prop.type \=== 'number') return 'float64';  
      if (prop.type \=== 'boolean') return 'bool';  
      if (prop.type \=== 'array') {  
        if (prop.items && prop.items.type \=== 'object') {  
          const subName \= \`\${toPascalCase(propKey)}Item\`;  
          subStructs.push(generateStruct(subName, prop.items));  
          return \`\[\]\${subName}\`;  
        }  
        return '\[\]string';  
      }  
      if (prop.type \=== 'object') {  
        const subName \= toPascalCase(propKey);  
        subStructs.push(generateStruct(subName, prop));  
        return subName;  
      }  
      return 'interface{}';  
    }

    function generateStruct(name, s) {  
      let code \= \`type \${name} struct {\\n\`;  
      const sReqs \= s.required || \[\];  
      const sProps \= s.properties || {};  
      for (const \[k, v\] of Object.entries(sProps)) {  
        const fieldName \= toPascalCase(k);  
        const goType \= parseProp(k, v);  
        const reqTag \= sReqs.includes(k) ? ' binding:"required"' : '';  
        code \+= \`\\t\${fieldName} \${goType} \\\`json:"\${k}"\${reqTag}\\\`\\n\`;  
      }  
      code \+= \`}\`;  
      return code;  
    }

    const reqs \= schema.required || \[\];  
    const props \= schema.properties || {};  
    let main \= \`type \${mainTitle} struct {\\n\`;  
    for (const \[k, v\] of Object.entries(props)) {  
      const fieldName \= toPascalCase(k);  
      const goType \= parseProp(k, v);  
      const reqTag \= reqs.includes(k) ? ' binding:"required"' : '';  
      main \+= \`\\t\${fieldName} \${goType} \\\`json:"\${k}"\${reqTag}\\\`\\n\`;  
    }  
    main \+= \`}\\n\`;

    if (subStructs.length \> 0\) {  
      out \+= subStructs.join('\\n\\n') \+ '\\n\\n';  
    }  
    out \+= main;  
    return out;  
  }

  return {  
    toZod,  
    toPydantic,  
    toTypeScript,  
    toGolang  
  };  
}));  

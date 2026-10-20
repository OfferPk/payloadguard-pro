/\*\*  
 \* PayloadGuard Pro \- JSON Schema Validation Engine  
 \* Zero-dependency client-side validator supporting Draft 7 and Draft 2020-12 constraints.  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.ValidatorEngine \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function() {

  function validate(schema, data, path \= '\$') {  
    const errors \= \[\];

    if (\!schema || typeof schema \!== 'object') {  
      return { valid: true, errors: \[\] };  
    }

    // Type validation  
    if (schema.type) {  
      const types \= Array.isArray(schema.type) ? schema.type : \[schema.type\];  
      let matchesType \= false;  
      const actualType \= getActualType(data);

      for (const t of types) {  
        if (t \=== 'integer' && Number.isInteger(data)) matchesType \= true;  
        else if (t \=== 'number' && typeof data \=== 'number' && \!isNaN(data)) matchesType \= true;  
        else if (t \=== actualType) matchesType \= true;  
      }

      if (\!matchesType && data \!== undefined) {  
        errors.push({  
          path,  
          keyword: 'type',  
          message: \`Expected type \${types.join(' | ')}, got \${actualType}\`,  
          expected: types.join(' | '),  
          actual: actualType  
        });  
      }  
    }

    // Enum validation  
    if (schema.enum && Array.isArray(schema.enum)) {  
      if (\!schema.enum.includes(data)) {  
        errors.push({  
          path,  
          keyword: 'enum',  
          message: \`Value "\${data}" is not in allowed enum: \[\${schema.enum.join(', ')}\]\`,  
          expected: schema.enum,  
          actual: data  
        });  
      }  
    }

    // Number constraints  
    if (typeof data \=== 'number') {  
      if (schema.minimum \!== undefined && data \< schema.minimum) {  
        errors.push({  
          path,  
          keyword: 'minimum',  
          message: \`Value \${data} is less than minimum \${schema.minimum}\`,  
          expected: \`\>= \${schema.minimum}\`,  
          actual: data  
        });  
      }  
      if (schema.maximum \!== undefined && data \> schema.maximum) {  
        errors.push({  
          path,  
          keyword: 'maximum',  
          message: \`Value \${data} exceeds maximum \${schema.maximum}\`,  
          expected: \`\<= \${schema.maximum}\`,  
          actual: data  
        });  
      }  
    }

    // String constraints  
    if (typeof data \=== 'string') {  
      if (schema.minLength \!== undefined && data.length \< schema.minLength) {  
        errors.push({  
          path,  
          keyword: 'minLength',  
          message: \`String length (\${data.length}) is shorter than minLength \${schema.minLength}\`,  
          expected: \`\>= \${schema.minLength}\`,  
          actual: data.length  
        });  
      }  
      if (schema.maxLength \!== undefined && data.length \> schema.maxLength) {  
        errors.push({  
          path,  
          keyword: 'maxLength',  
          message: \`String length (\${data.length}) exceeds maxLength \${schema.maxLength}\`,  
          expected: \`\<= \${schema.maxLength}\`,  
          actual: data.length  
        });  
      }  
      if (schema.pattern) {  
        try {  
          const re \= new RegExp(schema.pattern);  
          if (\!re.test(data)) {  
            errors.push({  
              path,  
              keyword: 'pattern',  
              message: \`String does not match required regex pattern: \${schema.pattern}\`,  
              expected: schema.pattern,  
              actual: data  
            });  
          }  
        } catch (e) {  
          // ignore invalid regex in schema  
        }  
      }  
      if (schema.format \=== 'email') {  
        const emailRe \= /^\[^\\s@\]+@\[^\\s@\]+\\.\[^\\s@\]+\$/;  
        if (\!emailRe.test(data)) {  
          errors.push({  
            path,  
            keyword: 'format',  
            message: \`Value "\${data}" is not a valid email address\`,  
            expected: 'RFC 5322 email format',  
            actual: data  
          });  
        }  
      }  
    }

    // Object constraints  
    if (typeof data \=== 'object' && data \!== null && \!Array.isArray(data)) {  
      if (schema.required && Array.isArray(schema.required)) {  
        for (const reqKey of schema.required) {  
          if (data\[reqKey\] \=== undefined) {  
            errors.push({  
              path: \`\${path}.\${reqKey}\`,  
              keyword: 'required',  
              message: \`Missing required property "\${reqKey}"\`,  
              expected: 'Present',  
              actual: 'Undefined'  
            });  
          }  
        }  
      }

      if (schema.properties) {  
        for (const \[propKey, propSchema\] of Object.entries(schema.properties)) {  
          if (data\[propKey\] \!== undefined) {  
            const sub \= validate(propSchema, data\[propKey\], \`\${path}.\${propKey}\`);  
            errors.push(...sub.errors);  
          }  
        }  
      }  
    }

    // Array constraints  
    if (Array.isArray(data)) {  
      if (schema.minItems \!== undefined && data.length \< schema.minItems) {  
        errors.push({  
          path,  
          keyword: 'minItems',  
          message: \`Array has \${data.length} items; minimum required is \${schema.minItems}\`,  
          expected: \`\>= \${schema.minItems}\`,  
          actual: data.length  
        });  
      }  
      if (schema.maxItems \!== undefined && data.length \> schema.maxItems) {  
        errors.push({  
          path,  
          keyword: 'maxItems',  
          message: \`Array has \${data.length} items; maximum allowed is \${schema.maxItems}\`,  
          expected: \`\<= \${schema.maxItems}\`,  
          actual: data.length  
        });  
      }  
      if (schema.items) {  
        data.forEach((item, idx) \=\> {  
          const sub \= validate(schema.items, item, \`\${path}\[\${idx}\]\`);  
          errors.push(...sub.errors);  
        });  
      }  
    }

    return {  
      valid: errors.length \=== 0,  
      errors  
    };  
  }

  function getActualType(val) {  
    if (val \=== null) return 'null';  
    if (Array.isArray(val)) return 'array';  
    return typeof val;  
  }

  return {  
    validate  
  };  
}));  

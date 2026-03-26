# Cambios Implementados: Reasignación de Funcionarios a Otros Centros de Salud

## Resumen
Se ha mejorado la funcionalidad de **reasignación de funcionarios a otros centros de salud** durante la revisión de dotación. Esto permite corregir casos donde un funcionario está registrado en un centro distinto al que le corresponde.

**Versión**: 2.1  
**Fecha de actualización**: 26 de marzo de 2026

## Cambios Realizados en esta Sesión

### 1. **Interfaz HTML del Modal - MEJORADA**
**Ubicación**: Sección "Reasignación a Centro de Salud" en el modal (`index.html` líneas ~500-530)

#### Cambios principales:
- ✅ Nuevo campo de solo lectura: **"Centro a Revisar (actual)"** para mostrar claramente cuál centro está siendo revisado
- ✅ Alerta rediseñada con clase `warn` en lugar de `info` para mejor visibilidad
- ✅ Texto más descriptivo: *"Este funcionario está registrado en un centro de salud diferente al que actualmente se está revisando"*
- ✅ Confirmación mejorada: Muestra el nombre del centro de destino dinámicamente con el elemento `#rm-reasign-destino`
- ✅ Estilos mejorados: Bordes y colores consistentes con el diseño de la aplicación
- ✅ Opción por defecto más clara: `"← Sin cambio, dejar en su centro actual"`

### 2. **Función `editarPersona(rutCompleto)` - MEJORADA**
**Ubicación**: `index.html` líneas ~1280-1320

#### Mejoras:
```javascript
// Nueva línea: Se captura el centro siendo revisado
document.getElementById('rm-cesfam-revisando').value=revCesfamActual;

// Mejora del dropdown: Filtra el centro actual para evitar duplicados
CENTROS.forEach(c=>{
  if(c!==cesfamActual){  // ← Solo muestra otros centros
    const opt=document.createElement('option');
    opt.value=c;
    opt.textContent=c;
    selReasign.appendChild(opt);
  }
});
```

**Beneficio**: El usuario no puede reasignar a un funcionario a su mismo centro (evita confusión y errores lógicos)

### 3. **Función `chequearReasign()` - COMPLETAMENTE REESCRITA**
**Ubicación**: `index.html` líneas ~1330-1350

#### Nueva implementación:
```javascript
function chequearReasign(){
  const cesfamActual=document.getElementById('rm-cesfam-actual').value;
  const cesfamRevisando=document.getElementById('rm-cesfam-revisando').value;
  const cesfamNuevo=document.getElementById('rm-cesfam-nuevo').value;
  const alertEl=document.getElementById('rm-alert-reasign');
  const infoEl=document.getElementById('rm-reasign-info');
  const destinoEl=document.getElementById('rm-reasign-destino');
  
  // Mostrar alerta si el funcionario no pertenece al centro siendo revisado
  const mostrarAlerta=cesfamActual!==cesfamRevisando&&!cesfamNuevo;
  alertEl.style.display=mostrarAlerta?'block':'none';
  
  // Mostrar info de confirmación si se seleccionó un nuevo centro
  const mostrarInfo=!!cesfamNuevo;
  infoEl.style.display=mostrarInfo?'block':'none';
  if(mostrarInfo&&destinoEl){
    destinoEl.textContent=cesfamNuevo;  // ← Actualiza dinámicamente
  }
}
```

**Mejora principal**: Ahora captura `cesfamRevisando` (el centro siendo revisado) en lugar de usar `revCesfamActual` directamente, permitiendo lógica más flexible y testeable.

### 4. **Tabla de Revisión - YA FUNCIONANDO**
**Ubicación**: Función `renderRevList()` líneas ~1210-1230

La tabla ya mostraba correctamente:
- ✅ Badge `✓ Reasignado` cuando un funcionario ha sido reasignado
- ✅ Tooltip con el nombre del nuevo centro: `title="Reasignado a ${rev.cesfam_nuevo}"`
- ✅ Diferenciación entre revisados, reasignados y pendientes

---

## Flujo Completo de Uso

### Escenario: "Revisar CESFAM AHUES pero encontrar un funcionario de CESFAM N°1"

1. **Abriendo el centro para revisar:**
   ```
   ✓ Se selecciona CESFAM AHUES
   ✓ Se carga la lista de sus funcionarios
   ```

2. **Editando funcionario "fuera de lugar":**
   ```
   ✓ Se abre modal de edición
   ✓ "Centro Actual": CESFAM N°1
   ✓ "Centro a Revisar": CESFAM AHUES
   ✓ Aparece alerta: "⚠ Este funcionario pertenece a otro centro..."
   ```

3. **Reasignando a centro correcto:**
   ```
   ✓ Usuario selecciona: "CESFAM AHUES" en el dropdown
   ✓ Alerta desaparece
   ✓ Aparece mensaje verde: "✓ Centro de destino seleccionado: CESFAM AHUES"
   ```

4. **Guardando cambios:**
   ```
   ✓ Se envía a Google Sheets con CESFAM=CESFAM AHUES
   ✓ Se registra REASIGNACION_CENTRO=Sí
   ✓ Se actualiza REVISION_MAP[rut].cesfam_nuevo=CESFAM AHUES
   ```

5. **Viendo resultados en tabla:**
   ```
   ✓ Badge cambia a: "✓ Reasignado" con tooltip
   ✓ El funcionario aparece en su nuevo centro
   ✓ Se puede hacer auditoría de cambios
   ```

---

## Datos Guardados en Google Sheets

| Campo | Valor Ejemplo |
|-------|----------------|
| `RUT` | `1234567` |
| `CESFAM` | `CESFAM AHUES` (nuevo centro) |
| `REASIGNACION_CENTRO` | `Sí` |
| `CARGO` | Se conserva |
| `Horas por contrato` | Se conserva |
| Otros datos | Se guardan normalmente |

---

## Interfaz Mejorada vs Anterior

### Antes (v2.0):
```
⚠ Alerta simple
Centro Actual: [________]
Reasignar a: [Dropdown simple]
Confirmación: Texto genérico
```

### Ahora (v2.1):
```
⚠ Centro Actual: [CESFAM N°1]  |  Centro a Revisar: [CESFAM AHUES]

Reasignar a Centro de Salud (si corresponde):
  [← Sin cambio, dejar en su centro actual ▼]
  ├─ CESFAM AHUES
  ├─ CESFAM SOFIA PINCHEIRA
  ├─ ...

✓ Centro de destino seleccionado:
  El funcionario será reasignado a CESFAM AHUES cuando guardes los cambios.
```

---

## Beneficios de las Mejoras

✅ **Claridad visual mejorada**: El usuario ve de un vistazo qué está pasando  
✅ **Prevención de errores**: No puede seleccionar el mismo centro  
✅ **Confirmación dinámica**: Muestra exactamente a dónde irá el funcionario  
✅ **Tracking completo**: Todos los cambios se registran  
✅ **Interfaz intuitiva**: Flujo claro de usuario  

---

## Pruebas Recomendadas

- [ ] Editar funcionario que pertenece a otro centro
- [ ] Verificar que la alerta aparece cuando debe
- [ ] Seleccionar un nuevo centro y confirmar el mensaje
- [ ] Guardar y verificar en Google Sheets
- [ ] Volver a la tabla y ver el badge `✓ Reasignado`
- [ ] Hacer clic en el badge para ver el tooltip
- [ ] Intentar reasignar múltiples funcionarios en una sesión

## Campos Guardados en Google Sheets

Se agrega un nuevo campo al registro de revisión:
- **`REASIGNACION_CENTRO`**: Indica si el funcionario fue reasignado ("Sí" o "No")

## Flujo de Uso

### Caso 1: Funcionario que pertenece a otro centro
1. Abrir un funcionario que no pertenece al centro siendo revisado
2. Ver la alerta: "⚠ Este funcionario pertenece a un centro diferente..."
3. Seleccionar el centro correcto del dropdown de reasignación
4. Confirmar el cambio al guardar

### Caso 2: Funcionario en el centro correcto
1. No se muestra alerta
2. El dropdown permanece con "Sin cambio" (predeterminado)
3. Se guarda la revisión normalmente

## Campos del Modal de Revisión

```
┌─────────────────────────────────────────┐
│ IDENTIFICACIÓN                          │
├─────────────────────────────────────────┤
│ RUT | Calidad Jurídica                  │
│ Nombre Funcionario                      │
├─────────────────────────────────────────┤
│ CLASIFICACIÓN                           │
├─────────────────────────────────────────┤
│ Descripción Planta | Cargo              │
│ Unidad Desempeño | Unidad Desempeño 2   │
│ Unidad Específica                       │
├─────────────────────────────────────────┤
│ REASIGNACIÓN A CENTRO DE SALUD  [NUEVO] │
├─────────────────────────────────────────┤
│ ⚠ Alerta (si aplica)                   │
│ Centro Actual: [campo solo lectura]     │
│ Reasignar a: [dropdown con todos]       │
│ ℹ Confirmación (si se selecciona)      │
├─────────────────────────────────────────┤
│ HORAS                                   │
├─────────────────────────────────────────┤
│ Horas por Contrato | Horas Totales      │
│ Modificaciones | Total Descuentos       │
│ Total Horas Clínicas                    │
├─────────────────────────────────────────┤
│ ENCOMENDACIONES                         │
├─────────────────────────────────────────┤
│ [10 dropdowns con asignaciones]         │
├─────────────────────────────────────────┤
│ OBSERVACIONES                           │
├─────────────────────────────────────────┤
│ [textarea para notas]                   │
└─────────────────────────────────────────┘
```

## Ventajas

✓ **Corrección en tiempo real**: Reasignar funcionarios durante la revisión
✓ **Claridad**: Alerta automática cuando hay desajuste de centro
✓ **Trazabilidad**: Se registra qué funcionarios fueron reasignados
✓ **Intuitivo**: Interfaz clara con confirmaciones visuales
✓ **Eficiencia**: No requiere recargar datos ni hacer cambios en múltiples places

## Notas Técnicas

- El dropdown de reasignación se puebla dinámicamente desde la variable `CENTROS`
- El almacenamiento local (`FUNC` y `REVISION_MAP`) se actualiza inmediatamente
- La persistencia en Google Sheets se maneja a través del endpoint `saveRevision`
- Las alertas se controlan mediante la función `chequearReasign()` que valida automáticamente el estado

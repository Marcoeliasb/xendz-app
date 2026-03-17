// ============================================================
// XENDZ BACKEND — Google Apps Script v3.0
// Reemplaza todo el código anterior con este completo
// ============================================================

const BASE_TAB    = 'Base';
const GUIAS_TAB   = 'Guias';
const FOLDER_NAME = 'Xendz Evidencias';
const TARIFA_GDL  = 600;
const TARIFA_MTY  = 750;
const LI_GDL      = 800;
const LI_MTY      = 1000;
const PROVEEDOR_CAJAS = { GDL: 'Autoexpress', MTY: 'Travisa' };
const PROVEEDOR_LI    = { GDL: 'Erik Santos',  MTY: 'Rosa Imelda' };
const PROVEEDOR_CDMX  = 'Erik Santos';

// ============================================================
// ROUTER PRINCIPAL
// ============================================================
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    let result;
    switch (data.action) {
      case 'finalizar':  result = finalizarServicio(data); break;
      case 'confirmar':  result = confirmarRecoleccion(data); break;
      default:           result = procesarRegistro(data);
    }
    return resp(200, result);
  } catch (err) {
    return resp(500, { error: err.message });
  }
}

function doGet(e) {
  try {
    const action = e.parameter.action;
    if (action === 'stats') return resp(200, getStats(e.parameter.proveedor));
    if (action === 'guias') return resp(200, getGuias(e.parameter.ciudad));
    return resp(200, { status: 'Xendz API activa', ts: new Date().toISOString() });
  } catch (err) {
    return resp(500, { error: err.message });
  }
}

// ============================================================
// 1. REGISTRO — escribe en Base (Middle Mile y LI)
// ============================================================
function procesarRegistro(data) {
  const ss    = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(BASE_TAB);

  if (!data.ciudad)   throw new Error('Campo requerido: ciudad');
  if (!data.concepto) throw new Error('Campo requerido: concepto');

  const fecha    = new Date(data.fecha || new Date());
  const mesNom   = getMes(fecha.getMonth());
  const ciudad   = data.ciudad.toUpperCase();
  const concepto = data.concepto.toUpperCase();
  const cantidad = parseInt(data.cantidad) || 1;
  const incluirCDMX = data.incluir_cdmx !== false;
  const filas = [];

  if (concepto === 'MM') {
    const tarifa = ciudad === 'GDL' ? TARIFA_GDL : TARIFA_MTY;
    const cobro  = cantidad * tarifa;

    // Foto recoleccion → Drive
    let fotoRecLink = '';
    if (data.foto_recoleccion) {
      try {
        const folder = getFolder(fecha);
        fotoRecLink = savePhoto(data.foto_recoleccion,
          'recoleccion_' + ciudad + '_' + fmtDate(fecha) + '.jpg', folder);
      } catch (e) { Logger.log('Foto rec error: ' + e); }
    }

    filas.push(buildFila(fecha, mesNom, 'Middle Mile', ciudad, 'Cajas',
      cantidad, PROVEEDOR_CAJAS[ciudad], calcCostoCajas(ciudad, cantidad), cobro, fotoRecLink));

    const provCiudad = data.proveedor_ciudad || getProveedorFleteCiudad(ciudad);
    filas.push(buildFila(fecha, mesNom, 'Middle Mile', ciudad, 'Flete Ciudad',
      cantidad, provCiudad, calcFleteCiudad(ciudad, cantidad), 0, ''));

    if (incluirCDMX) {
      const provCDMX = data.proveedor_cdmx || PROVEEDOR_CDMX;
      filas.push(buildFila(fecha, mesNom, 'Middle Mile', 'ND', 'Flete CDMX',
        cantidad, provCDMX, calcFleteCDMX(cantidad), 0, ''));
    }

  } else if (concepto === 'LI') {
    const tarifaLI = ciudad === 'GDL' ? LI_GDL : LI_MTY;
    const provLI   = data.proveedor_ciudad || PROVEEDOR_LI[ciudad];
    filas.push(buildFila(fecha, mesNom, 'Middle Mile', ciudad, 'Flete Ciudad',
      1, provLI, ciudad === 'GDL' ? 500 : 450, tarifaLI, ''));
  }

  filas.forEach(f => sheet.appendRow(f));
  return { ok: true, filas_escritas: filas.length, ciudad, concepto, cantidad,
           cdmx_incluido: incluirCDMX && concepto === 'MM',
           fecha: Utilities.formatDate(fecha, 'America/Mexico_City', 'dd/MM/yyyy') };
}

// ============================================================
// 2. FINALIZAR — guarda costos de fleteras + fotos de guías
// ============================================================
function finalizarServicio(data) {
  const ss    = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = getGuiasSheet(ss);
  const fecha = new Date(data.fecha || new Date());
  const folder = getFolder(fecha);
  const ids = [];

  for (const guia of (data.guias || [])) {
    let fotoGuiaLink = '';
    if (guia.foto_guia) {
      try { fotoGuiaLink = savePhoto(guia.foto_guia, 'guia_' + guia.ciudad + '_' + fmtDate(fecha) + '.jpg', folder); }
      catch (e) { Logger.log('Foto guia error: ' + e); }
    }

    const id = Utilities.getUuid();
    sheet.appendRow([
      id,
      fecha,
      guia.ciudad,
      guia.cajas,
      guia.costo,
      fotoGuiaLink,
      'pendiente',
      '',   // fecha confirmacion
      '',   // foto entrega
      '',   // firma
      data.proveedor_cdmx || PROVEEDOR_CDMX
    ]);
    ids.push(id);

    // Actualizar costo real en Base (columna Monto sin IVA de la fila de Cajas de hoy)
    actualizarCostoBase(ss, fecha, guia.ciudad, guia.costo);
  }

  return { ok: true, guias_creadas: ids.length, ids };
}

// ============================================================
// 3. CONFIRMAR — ciudad flete confirma recolección
// ============================================================
function confirmarRecoleccion(data) {
  const ss    = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(GUIAS_TAB);
  if (!sheet) throw new Error('Hoja Guias no encontrada');

  const rows = sheet.getDataRange().getValues();
  for (let i = 1; i < rows.length; i++) {
    if (String(rows[i][0]) === String(data.guia_id)) {
      const fecha  = new Date();
      const folder = getFolder(fecha);

      let fotoEntLink = '';
      if (data.foto_entrega) {
        try { fotoEntLink = savePhoto(data.foto_entrega, 'entrega_' + data.guia_id.slice(0,8) + '.jpg', folder); }
        catch (e) { Logger.log('Foto entrega error: ' + e); }
      }

      let firmaLink = '';
      if (data.firma) {
        try { firmaLink = savePhoto(data.firma, 'firma_' + data.guia_id.slice(0,8) + '.png', folder); }
        catch (e) { Logger.log('Firma error: ' + e); }
      }

      sheet.getRange(i + 1, 7).setValue('recogido');
      sheet.getRange(i + 1, 8).setValue(fecha);
      sheet.getRange(i + 1, 9).setValue(fotoEntLink);
      sheet.getRange(i + 1, 10).setValue(firmaLink);

      return { ok: true, guia_id: data.guia_id };
    }
  }
  throw new Error('Guía no encontrada: ' + data.guia_id);
}

// ============================================================
// 4. STATS — resumen semanal por proveedor
// ============================================================
function getStats(proveedor) {
  if (!proveedor) return { semana_pago: 0, hoy_pago: 0, semana_viajes: 0, semana_servicios: 0, historial: [] };

  const ss    = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(BASE_TAB);
  const data  = sheet.getDataRange().getValues();

  const hoy   = new Date(); hoy.setHours(0, 0, 0, 0);
  const lunes  = getLunes(hoy);

  const esCDMX = proveedor === 'Erik Santos' || proveedor === 'LalaMove';
  const esGDL  = proveedor.toLowerCase().includes('jose');
  const esMTY  = proveedor.toLowerCase().includes('rosa') || proveedor.toLowerCase().includes('eric rene');

  const histMap = {};
  let semPago = 0, hoyPago = 0, semViajes = 0, semServicios = 0;

  for (let i = 1; i < data.length; i++) {
    const row = data[i];
    if (!row[2]) continue;
    const fecha = new Date(row[2]); fecha.setHours(0, 0, 0, 0);
    if (fecha < lunes || fecha > hoy) continue;

    const prov    = String(row[7] || '');
    const concepto= String(row[5] || '');
    const costo   = parseFloat(row[8]) || 0;
    const ciudad  = String(row[4] || '');
    const cant    = parseFloat(row[6]) || 0;
    const fk      = Utilities.formatDate(fecha, 'America/Mexico_City', 'dd/MM');
    const esHoy   = fecha.getTime() === hoy.getTime();

    if (esCDMX && concepto === 'Flete CDMX' && prov === proveedor) {
      const pago = calcPagoCDMX(cant);
      semPago += pago; semViajes++;
      if (!histMap[fk]) histMap[fk] = { cajas: 0, viajes: 0, pago: 0 };
      histMap[fk].cajas  += cant;
      histMap[fk].viajes++;
      histMap[fk].pago   += pago;
      if (esHoy) hoyPago += pago;
    }

    if ((esGDL || esMTY) && concepto === 'Flete Ciudad' &&
        ((esGDL && ciudad === 'GDL') || (esMTY && ciudad === 'MTY')) && costo > 0) {
      semPago += costo; semServicios++;
      if (!histMap[fk]) histMap[fk] = { cajas: 0, pago: 0, tipo: '' };
      histMap[fk].cajas += cant;
      histMap[fk].pago  += costo;
      histMap[fk].tipo   = 'MM';
    }
  }

  const historial = Object.entries(histMap)
    .sort((a, b) => b[0].localeCompare(a[0]))
    .slice(0, 5)
    .map(([fecha, v]) => ({ fecha, ...v }));

  return { semana_pago: Math.round(semPago), hoy_pago: Math.round(hoyPago),
           semana_viajes: semViajes, semana_servicios: semServicios, historial };
}

// ============================================================
// 5. GUIAS — guías pendientes para ciudad flete
// ============================================================
function getGuias(ciudad) {
  if (!ciudad) return { guias: [] };
  const ss    = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName(GUIAS_TAB);
  if (!sheet) return { guias: [] };

  const rows  = sheet.getDataRange().getValues();
  const guias = [];

  for (let i = 1; i < rows.length; i++) {
    const [id, fecha, ciu, cajas, costo, fotoGuia, estado] = rows[i];
    if (String(ciu) === String(ciudad) && estado === 'pendiente' && id) {
      guias.push({
        id: String(id),
        fecha: Utilities.formatDate(new Date(fecha), 'America/Mexico_City', 'dd/MM/yyyy'),
        cajas: cajas,
        costo_proveedor: costo,
        foto_guia: String(fotoGuia || '')
      });
    }
  }

  return { guias };
}

// ============================================================
// TARIFARIOS
// ============================================================
function calcCostoCajas(ciudad, cantidad) {
  if (ciudad === 'GDL') return +(cantidad * 159.31).toFixed(2);
  if (cantidad <= 3)       return +(cantidad * 280.96).toFixed(2);
  else if (cantidad <= 6)  return +(cantidad * 271.29).toFixed(2);
  else if (cantidad <= 9)  return +(cantidad * 270.65).toFixed(2);
  else if (cantidad <= 12) return +(cantidad * 270.33).toFixed(2);
  else                     return +(cantidad * 270.09).toFixed(2);
}
function calcFleteCiudad(ciudad, cantidad) {
  if (ciudad === 'GDL') return 800;
  if (cantidad <= 6) return 700; if (cantidad <= 10) return 800; return 900;
}
function calcFleteCDMX(cantidad) {
  if (cantidad <= 11) return 700; if (cantidad <= 15) return 800;
  if (cantidad <= 16) return 900; return 1600;
}
function calcPagoCDMX(n) {
  if (n <= 11) return 700; if (n <= 15) return 800;
  if (n <= 16) return 900; return 1600;
}
function getProveedorFleteCiudad(ciudad) {
  return ciudad === 'GDL' ? 'Jose Luis Romero' : 'Rosa Imelda';
}

// ============================================================
// HELPERS — Drive, Sheet, Utils
// ============================================================
function getFolder(fecha) {
  const dateStr = Utilities.formatDate(fecha || new Date(), 'America/Mexico_City', 'yyyy-MM-dd');
  let root;
  const ri = DriveApp.getFoldersByName(FOLDER_NAME);
  root = ri.hasNext() ? ri.next() : DriveApp.createFolder(FOLDER_NAME);
  const si = root.getFoldersByName(dateStr);
  return si.hasNext() ? si.next() : root.createFolder(dateStr);
}

function savePhoto(dataUrl, filename, folder) {
  const base64 = dataUrl.includes(',') ? dataUrl.split(',')[1] : dataUrl;
  const mime   = dataUrl.includes('image/png') ? 'image/png' : 'image/jpeg';
  const blob   = Utilities.newBlob(Utilities.base64Decode(base64), mime, filename);
  const file   = folder.createFile(blob);
  file.setSharing(DriveApp.Access.ANYONE_WITH_LINK, DriveApp.Permission.VIEW);
  return 'https://drive.google.com/file/d/' + file.getId() + '/view';
}

function getGuiasSheet(ss) {
  let sheet = ss.getSheetByName(GUIAS_TAB);
  if (!sheet) {
    sheet = ss.insertSheet(GUIAS_TAB);
    sheet.appendRow(['ID','Fecha','Ciudad','Cajas','Costo Proveedor','Foto Guia',
                     'Estado','Fecha Confirmacion','Foto Entrega','Firma','Proveedor CDMX']);
    sheet.getRange(1, 1, 1, 11).setFontWeight('bold');
    sheet.setFrozenRows(1);
  }
  return sheet;
}

function actualizarCostoBase(ss, fecha, ciudad, costoReal) {
  const sheet = ss.getSheetByName(BASE_TAB);
  const data  = sheet.getDataRange().getValues();
  const fechaD = new Date(fecha); fechaD.setHours(0,0,0,0);

  for (let i = data.length - 1; i >= 1; i--) {
    if (!data[i][2]) continue;
    const rowFecha = new Date(data[i][2]); rowFecha.setHours(0,0,0,0);
    if (rowFecha.getTime() === fechaD.getTime() &&
        String(data[i][4]) === ciudad &&
        String(data[i][5]) === 'Cajas') {
      sheet.getRange(i + 1, 9).setValue(+(costoReal / 1.16).toFixed(2));
      break;
    }
  }
}

function buildFila(fecha, mes, proyecto, ciudad, concepto, cantidad, proveedor, costoSinIVA, cobro, fotoLink) {
  return [
    'CUBBO', mes, fecha, proyecto, ciudad, concepto, cantidad, proveedor,
    costoSinIVA,
    costoSinIVA > 0 && cantidad > 0 ? +(costoSinIVA / cantidad).toFixed(4) : 0,
    false, '', '', cobro || '', '', 'PENDIENTE', '', fotoLink || ''
  ];
}

function getMes(i) {
  return ['Enero','Febrero','Marzo','Abril','Mayo','Junio',
          'Julio','Agosto','Septiembre','Octubre','Noviembre','Diciembre'][i];
}
function fmtDate(d) { return Utilities.formatDate(d, 'America/Mexico_City', 'yyyyMMdd_HHmmss'); }
function getLunes(fecha) {
  const d = new Date(fecha);
  const dia = d.getDay();
  d.setDate(d.getDate() + (dia === 0 ? -6 : 1 - dia));
  d.setHours(0, 0, 0, 0);
  return d;
}
function resp(code, data) {
  return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}

// ============================================================
// FUNCIONES DE PRUEBA
// ============================================================
function testRegistro() {
  const r = procesarRegistro({ ciudad: 'GDL', concepto: 'MM', cantidad: 9,
    incluir_cdmx: true, proveedor_cdmx: 'Erik Santos', fecha: new Date().toISOString() });
  Logger.log(JSON.stringify(r));
}
function testFinalizar() {
  const r = finalizarServicio({ proveedor_cdmx: 'Erik Santos', fecha: new Date().toISOString(),
    guias: [{ ciudad: 'GDL', cajas: 9, costo: 1512, foto_guia: null }] });
  Logger.log(JSON.stringify(r));
}
function testGetGuias() {
  Logger.log(JSON.stringify(getGuias('GDL')));
}
function testGetStats() {
  Logger.log(JSON.stringify(getStats('Erik Santos')));
}

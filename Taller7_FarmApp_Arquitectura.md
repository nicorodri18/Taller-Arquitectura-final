# Taller 7 — Integración de Vistas de Arquitectura (FarmApp)

##  Objetivo
Integrar las vistas de **Negocio, Información, Aplicaciones, Infraestructura y Seguridad** en una narrativa visual y trazable que respalde los objetivos de FarmApp (cadena nacional de farmacias con e-commerce).

---

## 🗺️ Vista Integrada (Mapa en Capas)

> **Lectura:** cada etapa de negocio se alinea con entidades de datos, servicios/apps, componentes de infraestructura y controles de seguridad.

```mermaid
flowchart TB
  subgraph Negocio
    N1[Cliente]
    N2[Navega Catalogo]
    N3[Carga Prescripcion]
    N4[Validacion Rx]
    N5[Carrito]
    N6[Checkout y Pago]
    N7[Reserva de Stock]
    N8[Despacho y Entrega]
    N9[Rastreo y Notificaciones]
    N10[Posventa y CRM]
    N1 --> N2 --> N3 --> N4 --> N5 --> N6 --> N7 --> N8 --> N9 --> N10
  end

  subgraph Aplicaciones
    A1[App Movil]
    A2[Web E-commerce]
    A3[API Gateway]
    A4[MS Catalogo]
    A5[MS Prescripciones]
    A6[Motor de Reglas]
    A7[MS Carritos]
    A8[MS Pagos PSP]
    A9[MS Inventario]
    A10[MS Logistica TMS]
    A11[MS Notificaciones]
    A12[CRM Service Desk]
    A13[ETL DWH BI]
  end

  subgraph Infraestructura
    I1[CDN]
    I2[WAF]
    I3[K8s Front Back]
    I4[Object Storage KMS]
    I5[DB Replicada]
    I6[Message Bus Events]
    I7[VPC Privada]
    I8[SaaS PSP CRM]
    I9[Data Lake Warehouse]
  end

  subgraph Seguridad
    S1[TLS 1.3]
    S2[WAF Rules Rate Limiting]
    S3[MFA Staff RBAC]
    S4[Cifrado AES256]
    S5[Antifraude 3DS2]
    S6[Auditoria Inmutable]
    S7[DKIM SPF DMARC]
    S8[Anonimizacion BI]
    S9[DLP Basico]
  end

  %% Trazabilidad simplificada
  N2 --> A1
  N2 --> A2
  N2 --> A3
  A3 --> A4
  N3 --> A5
  A5 --> A6
  N5 --> A7
  N6 --> A8
  N7 --> A9
  N8 --> A10
  N9 --> A11
  N10 --> A12
  N10 --> A13
  A1 --> I1
  A2 --> I1
  I1 --> I2 --> I3
  A5 --> I4
  A9 --> I5
  A8 --> I8
  A10 --> I6
  A13 --> I9
  I1 --> S1
  I2 --> S2
  I3 --> S3
  I4 --> S4
  I8 --> S5
  I6 --> S6
  I9 --> S8
```


## 🔁 Proceso de Negocio (Compra con Prescripción)
```mermaid
flowchart LR
  C0[Inicio] --> C1[Explorar Catalogo]
  C1 --> C2{Producto con Rx?}
  C2 -- Si --> C3[Cargar Prescripcion]
  C2 -- No --> C4[Agregar al Carrito]
  C3 --> C5[Validacion Rx - Farmaceutico o Reglas]
  C5 -- Aprobada --> C4
  C5 -- Rechazada --> C6[Notificar y Sugerir Alternativas]
  C4 --> C7[Checkout y Pago]
  C7 --> C8[Reserva de Stock por Sucursal]
  C8 --> C9[Pick Pack Ship - Bodega]
  C9 --> C10[Despacho y Rastreo]
  C10 --> C11[Entrega]
  C11 --> C12[Posventa y CRM]

```



---

##  Modelo de Información (ER)
```mermaid
erDiagram
  Cliente ||--o{ Pedido : realiza
  Cliente ||--o{ Prescripcion : posee
  Prescripcion }o--|| Medico : emitida_por
  Farmaceutico ||--o{ Validacion : realiza
  Validacion }o--|| Prescripcion : sobre

  Pedido ||--|{ LineaPedido : contiene
  Producto ||--o{ LineaPedido : en
  Sucursal ||--o{ Inventario : gestiona
  Pedido }o--|| Pago : liquida
  Pedido }o--|| Entrega : genera
  Repartidor ||--o{ Entrega : realiza

  Cliente {
    string cliente_id PK
    string nombre
    string email
    string telefono
  }
  Producto {
    string sku PK
    string nombre
    string categoria
    float precio
  }
  Sucursal {
    string sucursal_id PK
    string ciudad
    string direccion
  }
  Inventario {
    string inventario_id PK
    string sku FK
    string sucursal_id FK
    int stock
    int stock_reservado
  }
  Pedido {
    string pedido_id PK
    string cliente_id FK
    date fecha
    string estado
    float total
    string descuento_id FK
  }
  LineaPedido {
    string linea_id PK
    string pedido_id FK
    string sku FK
    int cantidad
    float precio_unitario
  }
  Pago {
    string pago_id PK
    string pedido_id FK
    string metodo
    string estado
    string token_psp
  }
  Prescripcion {
    string rx_id PK
    string cliente_id FK
    string url_archivo
    date fecha_emision
  }
  Validacion {
    string val_id PK
    string rx_id FK
    string farmaceutico_id FK
    string resultado
    string comentario
  }
  Entrega {
    string entrega_id PK
    string pedido_id FK
    string direccion
    string estado
    string tracking
  }
```

---

##  Vista de Aplicaciones (Componentes)
```mermaid
flowchart TB
  UI1[App Movil] --> APIGW[API Gateway]
  UI2[Web E-commerce] --> APIGW

  subgraph Services
    CAT[MS Catalogo]
    RX[MS Prescripciones]
    RULES[Motor de Reglas]
    CART[MS Carritos]
    PAY[MS Pagos]
    INV[MS Inventario]
    LOG[MS Logistica TMS]
    NOTI[MS Notificaciones]
    CRM[CRM y Service Desk SaaS]
    ETL[ETL DWH BI]
  end

  APIGW --> CAT
  APIGW --> RX
  APIGW --> CART
  APIGW --> PAY
  APIGW --> INV
  APIGW --> LOG
  APIGW --> NOTI

  RX --> RULES
  PAY --> CRM
  NOTI --> CRM
  LOG --> CRM
  ETL --> CRM
```

---

##  Vista de Infraestructura (Despliegue híbrido)
```mermaid
flowchart LR
  subgraph Edge
    CDN[CDN] --> WAF[WAF]
  end

  WAF --> IAPIGW[API Gateway - K8s Ingress]

  subgraph Cloud_K8s
    FE[Frontend Pods]
    BE[Backend Pods]
    MQ[Message Bus / Events]
    OBJ[Object Storage + KMS]
    DWH[Data Lake / Warehouse]
  end

  IAPIGW --> FE
  IAPIGW --> BE
  BE --> MQ
  RXDB[DB Replicada\nmulti-sucursal]
  BE --> RXDB
  BE --> OBJ
  ETLJ[ETL Jobs] --> DWH
  SaaS[PSP / CRM SaaS]
```

---

##  Vista de Seguridad (Controles por capa)
- **Identidad y Acceso**: OIDC/OAuth2 para clientes; IAM con RBAC y **MFA** para personal (farmacéuticos, operadores, admins). Segregación de funciones (SoD).
- **Protección de Datos**: **TLS 1.3** en tránsito; cifrado en reposo (**AES‑256** con **KMS**). Pseudonimización/anonimización en analítica. DLP básico para archivos de Rx.
- **Pagos**: Alcance reducido **PCI DSS 4.0** mediante tokenización con PSP; **3DS2**; monitoreo antifraude (reglas + ML si aplica).
- **Aplicación**: WAF, rate limiting, validación de entrada, **OWASP ASVS**; secretos en **Secrets Manager**; logs firmados/append‑only.
- **Infra**: Escaneo de imágenes/CI, políticas de red zero‑trust, rotación de claves, hardening de K8s y nodos.
- **Cumplimiento local**: **Ley 1581 de 2012 (Habeas Data, CO)** para tratamiento de datos personales; consentimiento informado y derechos ARCO.

---

## 🔗 Matriz de Trazabilidad (vista rápida)
Consulta el CSV adjunto `FarmApp_Traceability.csv` para ver el cruce **Negocio ⇄ Datos ⇄ Apps ⇄ Infra ⇄ Seguridad**.

---

##  Cómo presentarlo en clase
1. Abre el TXT **FarmApp_Miro_Prompt.txt** y crea el tablero con 5 swimlanes.
2. Pega las etapas de negocio y repite el patrón de trazabilidad vertical por cada etapa.
3. Exporta una imagen/PDF del tablero y referéncialo en el informe.

---

## Checklist para la rúbrica (5.0)
- [ ] **Integración clara** entre las 5 capas con líneas de trazabilidad.
- [ ] **Aplicado al cliente real** (si adaptan FarmApp): decisiones explícitas (SAGA, tokenización, multi‑sucursal).
- [ ] **Narrativa**: por qué estas decisiones maximizan continuidad del servicio y seguridad.
- [ ] **Investigación**: mención de marcos/estándares (ver referencias).

---

## Referencias y buenas prácticas
- TOGAF Standard: https://www.opengroup.org/togaf
- ArchiMate 3.2: https://pubs.opengroup.org/architecture/archimate3-doc/
- C4 Model: https://c4model.com
- OWASP ASVS 4.0.3: https://owasp.org/www-project-application-security-verification-standard/
- NIST SP 800‑53 r5: https://csrc.nist.gov/publications/sp800-53
- NIST SP 800‑63‑3 (Identidad): https://pages.nist.gov/800-63-3/
- PCI DSS 4.0: https://www.pcisecuritystandards.org
- ISO/IEC 27001:2022: https://www.iso.org/standard/27001
- DAMA‑DMBOK2 (Data Mgmt): https://www.dama.org/content/body-knowledge

---

### Notas
- Sustituye/ajusta entidades y servicios si el **cliente real** del equipo difiere de FarmApp.
- Si no se usa prescripción, omite RX y la validación.

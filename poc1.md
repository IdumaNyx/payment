REPORT zsimulate_bapi_orders.

TYPES: 
  BEGIN OF ty_kna1,
    kunnr TYPE kunnr,
    name1 TYPE name1,
  END OF ty_kna1,

  BEGIN OF ty_knvv,
    kunnr TYPE kunnr,
    vkorg TYPE vkorg,
    vtweg TYPE vtweg,
  END OF ty_knvv,

  BEGIN OF ty_mara,
    matnr TYPE matnr,
  END OF ty_mara,

  BEGIN OF ty_mvke,
    matnr TYPE matnr,
    vkorg TYPE vkorg,
    vtweg TYPE vtweg,
  END OF ty_mvke.

DATA: 
  lt_kna1 TYPE TABLE OF ty_kna1,
  lt_knvv TYPE TABLE OF ty_knvv,
  lt_mara TYPE TABLE OF ty_mara,
  lt_mvke TYPE TABLE OF ty_mvke,
  lv_ord_no TYPE i.

FIELD-SYMBOLS: 
  <fs_kna1> TYPE ty_kna1,
  <fs_knvv> TYPE ty_knvv,
  <fs_mara> TYPE ty_mara,
  <fs_mvke> TYPE ty_mvke.

PARAMETERS: 
  p_ord_no TYPE i DEFAULT 5.

START-OF-SELECTION.

" Get Customers
SELECT kunnr, name1 
  INTO TABLE lt_kna1 
  FROM kna1 
  UP TO 100 ROWS.

IF sy-subrc <> 0.
  WRITE: / 'No customers found!'.
  EXIT.
ENDIF.

" Get Customers with Sales Org and Distribution Channel
SELECT kunnr, vkorg, vtweg 
  INTO TABLE lt_knvv 
  FROM knvv
  FOR ALL ENTRIES IN lt_kna1
  WHERE kunnr = lt_kna1-kunnr.

IF sy-subrc <> 0.
  WRITE: / 'No valid customers with sales org and dist. channel!'.
  EXIT.
ENDIF.

" Get Materials
SELECT matnr 
  INTO TABLE lt_mara 
  FROM mara 
  UP TO 100 ROWS.

IF sy-subrc <> 0.
  WRITE: / 'No materials found!'.
  EXIT.
ENDIF.

" Get Materials with Sales Org and Distribution Channel
SELECT matnr, vkorg, vtweg 
  INTO TABLE lt_mvke 
  FROM mvke
  FOR ALL ENTRIES IN lt_mara
  WHERE matnr = lt_mara-matnr.

IF sy-subrc <> 0.
  WRITE: / 'No materials found in MVKE table!'.
  EXIT.
ENDIF.

" Start Creating Orders
DATA: ls_bapisdhd1 TYPE bapisdhd1,
      lt_bapisditem TYPE TABLE OF bapisditem,
      lt_bapipartnr TYPE TABLE OF bapipartnr,
      lt_bapireturn TYPE TABLE OF bapiret2,
      lv_salesorder TYPE vbeln_va.

DO p_ord_no TIMES.

  " Select a random customer
  READ TABLE lt_knvv ASSIGNING <fs_knvv> INDEX sy-index MOD LINES( lt_knvv ) + 1.
  IF sy-subrc <> 0.
    WRITE: / 'No valid customer found, skipping order'.
    CONTINUE.
  ENDIF.

  " Select a random material
  READ TABLE lt_mvke ASSIGNING <fs_mvke> INDEX sy-index MOD LINES( lt_mvke ) + 1.
  IF sy-subrc <> 0.
    WRITE: / 'No valid material found, skipping order'.
    CONTINUE.
  ENDIF.

  " Populate Sales Order Header
  ls_bapisdhd1-doc_type = 'TA'. " Standard Order Type
  ls_bapisdhd1-sales_org = <fs_knvv>-vkorg.
  ls_bapisdhd1-distr_chan = <fs_knvv>-vtweg.
  ls_bapisdhd1-division = '00'.

  " Populate Sales Order Item
  APPEND INITIAL LINE TO lt_bapisditem ASSIGNING FIELD-SYMBOL(<fs_item>).
  <fs_item>-itme_no = '10'.
  <fs_item>-material = <fs_mvke>-matnr.
  <fs_item>-target_qty = '1'.

  " Populate Partner Data
  APPEND INITIAL LINE TO lt_bapipartnr ASSIGNING FIELD-SYMBOL(<fs_partnr>).
  <fs_partnr>-partn_role = 'AG'. " Sold-To Party
  <fs_partnr>-partn_numb = <fs_knvv>-kunnr.

  " Call BAPI to create order
  CALL FUNCTION 'BAPI_SALESORDER_CREATEFROMDAT2'
    EXPORTING
      order_header_in = ls_bapisdhd1
    TABLES
      order_items_in  = lt_bapisditem
      order_partners  = lt_bapipartnr
      return          = lt_bapireturn.

  " Check if order is created successfully
  READ TABLE lt_bapireturn WITH KEY type = 'E' TRANSPORTING NO FIELDS.
  IF sy-subrc = 0.
    WRITE: / 'Failed to create order for', <fs_knvv>-kunnr, 'Material:', <fs_mvke>-matnr.
    CONTINUE.
  ENDIF.

  " Commit the BAPI
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING
      wait = 'X'.

  " Retrieve Sales Order Number
  READ TABLE lt_bapireturn WITH KEY type = 'S' TRANSPORTING lv_salesorder.
  IF sy-subrc = 0.
    WRITE: / 'Created Sales Order:', lv_salesorder, 'Customer:', <fs_knvv>-kunnr, 'Material:', <fs_mvke>-matnr.
  ELSE.
    WRITE: / 'Sales Order creation failed!'.
  ENDIF.

ENDDO.


-----------‐----------------------–--------------'

REPORT zsimulate_bapi_orders.

TYPES: 
  BEGIN OF ty_kna1,
    kunnr TYPE kunnr,
    name1 TYPE name1,
  END OF ty_kna1,

  BEGIN OF ty_knvv,
    kunnr TYPE kunnr,
    vkorg TYPE vkorg,
    vtweg TYPE vtweg,
  END OF ty_knvv,

  BEGIN OF ty_mara,
    matnr TYPE matnr,
  END OF ty_mara,

  BEGIN OF ty_mvke,
    matnr TYPE matnr,
    vkorg TYPE vkorg,
    vtweg TYPE vtweg,
  END OF ty_mvke.

DATA: 
  lt_kna1 TYPE TABLE OF ty_kna1,
  lt_knvv TYPE TABLE OF ty_knvv,
  lt_mara TYPE TABLE OF ty_mara,
  lt_mvke TYPE TABLE OF ty_mvke,
  lv_ord_no TYPE i.

PARAMETERS: 
  p_ord_no TYPE i DEFAULT 5.

START-OF-SELECTION.

" Get Customers
SELECT kunnr, name1 
  INTO TABLE lt_kna1 
  FROM kna1 
  UP TO 100 ROWS.

IF sy-subrc <> 0.
  WRITE: / 'No customers found!'.
  EXIT.
ENDIF.

" Get Customers with Sales Org and Distribution Channel
SELECT kunnr, vkorg, vtweg 
  INTO TABLE lt_knvv 
  FROM knvv
  FOR ALL ENTRIES IN lt_kna1
  WHERE kunnr = lt_kna1-kunnr.

IF sy-subrc <> 0.
  WRITE: / 'No valid customers with sales org and dist. channel!'.
  EXIT.
ENDIF.

" Get Materials
SELECT matnr 
  INTO TABLE lt_mara 
  FROM mara 
  UP TO 100 ROWS.

IF sy-subrc <> 0.
  WRITE: / 'No materials found!'.
  EXIT.
ENDIF.

" Get Materials with Sales Org and Distribution Channel
SELECT matnr, vkorg, vtweg 
  INTO TABLE lt_mvke 
  FROM mvke
  FOR ALL ENTRIES IN lt_mara
  WHERE matnr = lt_mara-matnr.

IF sy-subrc <> 0.
  WRITE: / 'No materials found in MVKE table!'.
  EXIT.
ENDIF.

" Start Creating Orders
DATA: ls_bapisdhd1 TYPE bapisdhd1,
      lt_bapisditem TYPE TABLE OF bapisditem,
      lt_bapipartnr TYPE TABLE OF bapipartnr,
      lt_bapireturn TYPE TABLE OF bapiret2,
      lv_salesorder TYPE vbeln_va.

DATA: ls_knvv TYPE ty_knvv,
      ls_mvke TYPE ty_mvke,
      ls_bapireturn TYPE bapiret2.

DO p_ord_no TIMES.

  " Select a random customer
  READ TABLE lt_knvv INTO ls_knvv INDEX sy-index MOD LINES( lt_knvv ) + 1.
  IF sy-subrc <> 0.
    WRITE: / 'No valid customer found, skipping order'.
    CONTINUE.
  ENDIF.

  " Select a random material
  READ TABLE lt_mvke INTO ls_mvke INDEX sy-index MOD LINES( lt_mvke ) + 1.
  IF sy-subrc <> 0.
    WRITE: / 'No valid material found, skipping order'.
    CONTINUE.
  ENDIF.

  " Populate Sales Order Header
  ls_bapisdhd1-doc_type = 'TA'. " Standard Order Type
  ls_bapisdhd1-sales_org = ls_knvv-vkorg.
  ls_bapisdhd1-distr_chan = ls_knvv-vtweg.
  ls_bapisdhd1-division = '00'.

  " Populate Sales Order Item
  CLEAR lt_bapisditem.
  APPEND INITIAL LINE TO lt_bapisditem ASSIGNING FIELD-SYMBOL(<fs_item>).
  <fs_item>-itme_no = '10'.
  <fs_item>-material = ls_mvke-matnr.
  <fs_item>-target_qty = '1'.

  " Populate Partner Data
  CLEAR lt_bapipartnr.
  APPEND INITIAL LINE TO lt_bapipartnr ASSIGNING FIELD-SYMBOL(<fs_partnr>).
  <fs_partnr>-partn_role = 'AG'. " Sold-To Party
  <fs_partnr>-partn_numb = ls_knvv-kunnr.

  " Simulate BAPI before creating order
  CLEAR lt_bapireturn.
  CALL FUNCTION 'BAPI_SALESORDER_SIMULATE'
    EXPORTING
      order_header_in = ls_bapisdhd1
    TABLES
      order_items_in  = lt_bapisditem
      order_partners  = lt_bapipartnr
      return          = lt_bapireturn.

  " Check if simulation is successful
  READ TABLE lt_bapireturn INTO ls_bapireturn WITH KEY type = 'E'.
  IF sy-subrc = 0.
    WRITE: / 'Simulation failed for Customer:', ls_knvv-kunnr, 'Material:', ls_mvke-matnr.
    CONTINUE.
  ENDIF.

  " Call BAPI to create order
  CLEAR lt_bapireturn.
  CALL FUNCTION 'BAPI_SALESORDER_CREATEFROMDAT2'
    EXPORTING
      order_header_in = ls_bapisdhd1
    TABLES
      order_items_in  = lt_bapisditem
      order_partners  = lt_bapipartnr
      return          = lt_bapireturn.

  " Check if order is created successfully
  READ TABLE lt_bapireturn INTO ls_bapireturn WITH KEY type = 'E'.
  IF sy-subrc = 0.
    WRITE: / 'Failed to create order for', ls_knvv-kunnr, 'Material:', ls_mvke-matnr.
    CONTINUE.
  ENDIF.

  " Commit the BAPI
  CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
    EXPORTING
      wait = 'X'.

  " Retrieve Sales Order Number
  READ TABLE lt_bapireturn INTO ls_bapireturn WITH KEY type = 'S'.
  IF sy-subrc = 0.
    WRITE: / 'Created Sales Order:', lv_salesorder, 'Customer:', ls_knvv-kunnr, 'Material:', ls_mvke-matnr.
  ELSE.
    WRITE: / 'Sales Order creation failed!'.
  ENDIF.

ENDDO.
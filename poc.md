REPORT ZSIMULATE_BAPI_ORDERS.

PARAMETERS: p_ord_no TYPE i DEFAULT 10.

DATA: 
  lt_kna1 TYPE TABLE OF kna1, 
  lt_knvv TYPE TABLE OF knvv, 
  lt_mara TYPE TABLE OF mara, 
  lt_mvke TYPE TABLE OF mvke, 
  lt_orders TYPE TABLE OF bapisdhead, 
  lt_items TYPE TABLE OF bapiitemin, 
  lt_partners TYPE TABLE OF bapipartnr, 
  lt_return TYPE TABLE OF bapiret2, 
  lt_selected_orders TYPE TABLE OF bapisdhead.

DATA: 
  ls_kna1 TYPE kna1, 
  ls_knvv TYPE knvv, 
  ls_mara TYPE mara, 
  ls_mvke TYPE mvke, 
  ls_order TYPE bapisdhead, 
  ls_item TYPE bapiitemin, 
  ls_partner TYPE bapipartnr, 
  lv_return TYPE bapiret2, 
  lv_vkorg TYPE vkorg, 
  lv_vtweg TYPE vtweg, 
  lv_spart TYPE spart, 
  lv_index TYPE i, 
  lv_matnr TYPE matnr, 
  lv_kunnr TYPE kunnr.

" Step 1: Fetch only customers that have Sales Org and Dist. Channel
SELECT kunnr, land1, name1 
  INTO TABLE lt_kna1 
  FROM kna1
  WHERE kunnr IN (SELECT kunnr FROM knvv)
  UP TO 100 ROWS.

" Step 2: Fetch Sales Org and Distribution Channel data
SELECT kunnr, vkorg, vtweg, spart
  INTO TABLE lt_knvv
  FROM knvv
  WHERE kunnr IN (SELECT kunnr FROM kna1)
  UP TO 100 ROWS.

" Step 3: Fetch Material Master Data
SELECT matnr, matkl 
  INTO TABLE lt_mara 
  FROM mara 
  UP TO 100 ROWS.

" Step 4: Fetch Material Sales Data for matching Sales Org and Dist. Channel
SELECT matnr, vkorg, vtweg
  INTO TABLE lt_mvke
  FROM mvke
  WHERE vkorg IN (SELECT vkorg FROM knvv)
  AND vtweg IN (SELECT vtweg FROM knvv)
  UP TO 100 ROWS.

DO p_ord_no TIMES.

  " Select a random customer
  IF LINES( lt_kna1 ) > 0.
    lv_index = sy-index MOD LINES( lt_kna1 ) + 1.
    READ TABLE lt_kna1 INTO ls_kna1 INDEX lv_index.
  ELSE.
    CONTINUE.
  ENDIF.

  " Get Sales Org, Dist. Channel & Division
  READ TABLE lt_knvv INTO ls_knvv WITH KEY kunnr = ls_kna1-kunnr.
  IF sy-subrc <> 0.
    WRITE: / 'Skipping customer (No Sales Org/Dist. Channel):', ls_kna1-kunnr.
    CONTINUE.
  ENDIF.

  lv_vkorg = ls_knvv-vkorg.
  lv_vtweg = ls_knvv-vtweg.
  lv_spart = ls_knvv-spart.

  " Select a random material
  IF LINES( lt_mara ) > 0.
    lv_index = sy-index MOD LINES( lt_mara ) + 1.
    READ TABLE lt_mara INTO ls_mara INDEX lv_index.
    READ TABLE lt_mvke INTO ls_mvke WITH KEY matnr = ls_mara-matnr.
    IF sy-subrc <> 0.
      WRITE: / 'Skipping material:', ls_mara-matnr.
      CONTINUE.
    ENDIF.
    lv_matnr = ls_mara-matnr.
  ELSE.
    CONTINUE.
  ENDIF.

  " Ensure order is not duplicated
  READ TABLE lt_selected_orders WITH KEY sales_org = lv_vkorg TRANSPORTING NO FIELDS.
  IF sy-subrc = 0.
    CONTINUE.
  ENDIF.

  " Create Order Header
  CLEAR ls_order.
  ls_order-doc_type = 'TA'.
  ls_order-sales_org = lv_vkorg.
  ls_order-distr_chan = lv_vtweg.
  ls_order-division = lv_spart.

  APPEND ls_order TO lt_orders.

  " Create Order Item
  CLEAR ls_item.
  ls_item-material = lv_matnr.
  ls_item-plant = '1000'.
  ls_item-TARGET_QTY = 1 + sy-index MOD 10.

  APPEND ls_item TO lt_items.

  " Create Partner Data
  CLEAR ls_partner.
  ls_partner-partn_role = 'AG'.
  ls_partner-partn_numb = ls_kna1-kunnr.

  APPEND ls_partner TO lt_partners.

  APPEND ls_order TO lt_selected_orders.

  " Call BAPI to Simulate Sales Order
  CALL FUNCTION 'BAPI_SALESORDER_SIMULATE'
    EXPORTING
      order_header_in = ls_order
    TABLES
      order_items_in  = lt_items
      order_partners  = lt_partners
      messagetable    = lt_return.

  " Display Results
  LOOP AT lt_return INTO lv_return.
    WRITE: / lv_return-type, lv_return-id, lv_return-number, lv_return-message.
  ENDLOOP.

ENDDO.
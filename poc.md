REPORT ZSIMULATE_BAPI_ORDERS.

PARAMETERS: p_ord_no TYPE i DEFAULT 10.

DATA: lt_kna1 TYPE TABLE OF kna1,
      lt_knvv TYPE TABLE OF knvv,
      lt_knvp TYPE TABLE OF knvp,
      lt_mara TYPE TABLE OF mara,
      lt_mvke TYPE TABLE OF mvke,
      lt_vbak TYPE TABLE OF vbak,
      lt_vbap TYPE TABLE OF vbap,
      lt_orders TYPE TABLE OF bapisdhead,
      lt_items TYPE TABLE OF bapiitemin,
      lt_partners TYPE TABLE OF bapipartnr,
      lt_return TYPE TABLE OF bapiret2,
      lt_return1 TYPE TABLE OF bapiret2.

DATA: ls_kna1 TYPE kna1,
      ls_knvv TYPE knvv,
      ls_knvp TYPE knvp,
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
      lv_kunnr TYPE kunnr,
      lv_random TYPE i,
      lt_selected_orders TYPE TABLE OF bapisdhead.


"SELECT kunnr, vkorg, vtweg FROM knvv INTO TABLE lt_knvv FOR ALL ENTRIES IN lt_kna1 WHERE kunnr = lt_kna1-kunnr.
SELECT mandt, kunnr, land1, name1 FROM kna1 INTO TABLE @lt_kna1." UP TO 100 ROWS.
SELECT  knvv~kunnr, knvv~vkorg, knvv~vtweg, knvv~spart, kna1~land1, kna1~name1 INTO TABLE @lt_knvv UP TO 100 ROWS FROM knvv INNER JOIN kna1 ON knvv~kunnr = kna1~kunnr.
"SELECT mandt, kunnr, vkorg, vtweg, spart FROM knvv INTO TABLE @lt_knvv FOR ALL ENTRIEs IN @lt_kna1 WHERE kunnr = @lt_kna1-kunnr.
SELECT matnr, matkl FROM mara INTO TABLE @lt_mara." UP TO 100 ROWS.
SELECT matnr, vkorg, vtweg FROM mvke INTO TABLE @lt_mvke FOR ALL ENTRIES IN @lt_mara WHERE matnr = @lt_mara-matnr.

*LOOP AT lt_knvv INTO DATA(c).
*    WRITE: / c-mandt, c-kunnr, c-vkorg, c-vtweg, c-spart.
*ENDLOOP.


DO p_ord_no TIMES.

  IF LINES( lt_kna1 ) > 0.
    lv_index = sy-index MOD LINES( lt_kna1 ) + 1.
    READ TABLE lt_kna1 INTO ls_kna1 INDEX lv_index.
  ELSE.
    "CONTINUE.
  ENDIF.

  READ TABLE lt_knvv INTO ls_knvv WITH KEY kunnr = ls_kna1-kunnr.
    IF sy-subrc = 0.
        lv_vkorg = ls_knvv-vkorg.
        lv_vtweg = ls_knvv-vtweg.
        lv_spart = ls_knvv-spart.
    ELSE.
        WRITE: / 'No sales org or distr. channel for :', ls_kna1-kunnr.
        "CONTINUE.
    ENDIF.

  IF LINES( lt_mara ) > 0.
   lv_index = sy-index + 1 MOD LINES( lt_mara ).
   READ TABLE lt_mara INTO ls_mara INDEX lv_index.
   READ TABLE lt_mvke INTO ls_mvke WITH KEY matnr = ls_mara-matnr.
   lv_matnr = ls_mara-matnr.
  ELSE.
    "CONTINUE.
  ENDIF.



 " IF NOT LINE_EXISTS( lt_selected_orders[ sales_org = ls_knvv-vkorg ] ).

  CLEAR ls_order.
  ls_order-doc_type = 'TA'.
  ls_order-sales_org = lv_vkorg.
  ls_order-distr_chan = lv_vtweg.
  ls_order-division = lv_spart.

  APPEND ls_order TO lt_orders.

  CLEAR ls_item.
  CLEAR lt_items.
  ls_item-material = lv_matnr.
  ls_item-plant = '1000'.
  ls_item-TARGET_QTY = 1 + sy-index MOD 10.

  APPEND ls_item TO lt_items.

  CLEAR ls_partner.
  CLEAR lt_partners.
  ls_partner-partn_role = 'AG'.
  ls_partner-partn_numb = ls_kna1-kunnr.

  APPEND ls_partner TO lt_partners.

  APPEND ls_order TO lt_selected_orders.



CALL FUNCTION 'BAPI_SALESORDER_SIMULATE'
    EXPORTING
        order_header_in = ls_order
    TABLES
        order_items_in = lt_items
        order_partners = lt_partners
        messagetable = lt_return.
" ENDIF.

LOOP AT lt_return INTO lv_return.
    WRITE: / lv_return-type, lv_return-id, lv_return-number, lv_return-message.
ENDLOOP.
*WRITE: / 'Order:', ls_kna1-kunnr, 'Sales Org:', lv_vkorg, 'Dist. Channel:', lv_vtweg, 'Material:', lv_matnr.
*WRITE: / 'Order:', ls_order-sales_org, ls_order-distr_chan.

" WRITE: / 'Order:', ls_order-sales_org, ls_order-distr_chan, ls_item-material.
ENDDO.

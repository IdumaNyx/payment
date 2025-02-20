REPORT zsimulate_sales_order.

* Custom Types Definition
TYPES: BEGIN OF ty_vbak,
         sel    TYPE char1, " Selection Column
         vbeln  TYPE vbak-vbeln,
         kunnr  TYPE vbak-kunnr,
         erdat  TYPE dats,
         ernam  TYPE vbak-ernam,
         netwr  TYPE vbak-netwr,
       END OF ty_vbak.

TYPES: ty_vbak_tab TYPE TABLE OF ty_vbak.

TYPES: BEGIN OF ty_vbap,
         vbeln  TYPE vbap-vbeln,
         posnr  TYPE vbap-posnr,
         matnr  TYPE vbap-matnr,
         kwmeng TYPE vbap-kwmeng,
         netpr  TYPE vbap-netpr,
       END OF ty_vbap.

TYPES: ty_vbap_tab TYPE TABLE OF ty_vbap.

* Selection Screen
PARAMETERS: p_vbeln TYPE vbak-vbeln.

* Internal Tables
DATA lt_vbak TYPE ty_vbak_tab.
DATA lt_vbap TYPE ty_vbap_tab.

START-OF-SELECTION.

  " Fetch Sales Order Header Data
  SELECT vbeln, kunnr, erdat, ernam, netwr
    FROM vbak
    INTO TABLE lt_vbak
    WHERE vbeln = p_vbeln.

  IF lt_vbak IS INITIAL.
    MESSAGE 'Sales Order not found!' TYPE 'E'.
  ENDIF.

  " Fetch Sales Order Items
  SELECT vbeln, posnr, matnr, kwmeng, netpr
    FROM vbap
    INTO TABLE lt_vbap
    FOR ALL ENTRIES IN lt_vbak
    WHERE vbeln = lt_vbak-vbeln.

  " Display Data in ALV with Selection Option
  TRY.
      DATA lo_alv TYPE REF TO cl_salv_table.
      cl_salv_table=>factory( IMPORTING r_salv_table = lo_alv CHANGING t_table = lt_vbak ).
      
      " Enable selection column
      DATA(lo_columns) = lo_alv->get_columns( ).
      DATA(lo_column)  = lo_columns->get_column( 'SEL' ). " Selection column
      lo_column->set_short_text( 'Select' ).
      lo_column->set_medium_text( 'Select' ).
      lo_column->set_long_text( 'Select Order' ).

      lo_alv->display( ).

  CATCH cx_salv_msg.
      MESSAGE 'Error displaying ALV' TYPE 'E'.
  ENDTRY.

  " Capture user selection after ALV execution
  READ TABLE lt_vbak INTO DATA(ls_selected) WITH KEY sel = 'X'.
  IF sy-subrc <> 0.
    MESSAGE 'No order selected!' TYPE 'E'.
  ENDIF.

  " Display the selected order
  MESSAGE |Selected Order: { ls_selected-vbeln }| TYPE 'I'.

  " Prepare BAPI input
  DATA: ls_order_header    TYPE bapisdh1,
        lt_order_items     TYPE TABLE OF bapisditm,
        ls_order_item      TYPE bapisditm,
        lt_order_schedules TYPE TABLE OF bapischdl,
        ls_order_schedule  TYPE bapischdl,
        lt_order_partners  TYPE TABLE OF bapiparnr,
        ls_order_partner   TYPE bapiparnr,
        lt_return          TYPE TABLE OF bapiret2.

  " Populate Order Header
  ls_order_header-doc_type = 'TA'. " Standard Order
  ls_order_header-sales_org = '1000'. " Example Sales Org
  ls_order_header-distr_chan = '10'. " Example Distribution Channel
  ls_order_header-division = '00'. " Example Division

  " Populate Order Items
  LOOP AT lt_vbap INTO DATA(ls_vbap).
    CLEAR ls_order_item.
    ls_order_item-itm_number = ls_vbap-posnr.
    ls_order_item-material = ls_vbap-matnr.
    ls_order_item-order_qty = ls_vbap-kwmeng.
    APPEND ls_order_item TO lt_order_items.

    " Populate Schedule Lines
    CLEAR ls_order_schedule.
    ls_order_schedule-itm_number = ls_vbap-posnr.
    ls_order_schedule-sched_line = 1.
    ls_order_schedule-req_qty = ls_vbap-kwmeng.
    APPEND ls_order_schedule TO lt_order_schedules.
  ENDLOOP.

  " Populate Partner Data (Sold-to Party)
  ls_order_partner-partn_role = 'AG'. " Sold-to Party
  ls_order_partner-partn_numb = ls_selected-kunnr.
  APPEND ls_order_partner TO lt_order_partners.

  " Call BAPI to Simulate Sales Order
  CALL FUNCTION 'BAPI_SALESORDER_SIMULATE'
    EXPORTING
      order_header_in    = ls_order_header
    TABLES
      return             = lt_return
      order_items_in     = lt_order_items
      order_schedules_in = lt_order_schedules
      order_partners     = lt_order_partners.

  " Display Simulation Messages
  LOOP AT lt_return INTO DATA(ls_return).
    MESSAGE |{ ls_return-type }: { ls_return-message }| TYPE 'I'.
  ENDLOOP.
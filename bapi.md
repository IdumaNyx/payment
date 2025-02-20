REPORT zcreate_sales_order.

* Custom Types Definition
TYPES: BEGIN OF ty_vbak,
         sel    TYPE char1, " Selection Column
         vbeln  TYPE vbak-vbeln,
         kunnr  TYPE vbak-kunnr,
         erdat  TYPE vbak-erdat,
         ernam  TYPE vbak-ernam,
         netwr  TYPE vbak-netwr,
       END OF ty_vbak.

TYPES: ty_vbak_tab TYPE TABLE OF ty_vbak.

* Selection Screen
PARAMETERS: p_vbeln TYPE vbak-vbeln.

* Internal Table to Store Data
DATA lt_vbak TYPE ty_vbak_tab.

START-OF-SELECTION.

  " Fetch Sales Order Header Data
  SELECT vbeln, kunnr, erdat, ernam, netwr
    FROM vbak
    INTO TABLE lt_vbak
    WHERE vbeln = p_vbeln.

  IF lt_vbak IS INITIAL.
    MESSAGE 'Sales Order not found!' TYPE 'E'.
  ENDIF.

  " Add Selection Column
  LOOP AT lt_vbak ASSIGNING FIELD-SYMBOL(<fs_vbak>).
    <fs_vbak>-sel = space. " Default: No selection
  ENDLOOP.

  " Display Data in ALV with Selection Option
  TRY.
      DATA(lo_alv) = cl_salv_table=>factory( IMPORTING r_salv_table = lo_alv CHANGING t_table = lt_vbak ).
      
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

  " Display the selected order (for validation before creating a new one)
  MESSAGE |Selected Order: { ls_selected-vbeln }| TYPE 'I'.
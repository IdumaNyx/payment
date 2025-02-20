REPORT zcreate_sales_order.

* Custom Types Definition
TYPES: BEGIN OF ty_vbak,
         vbeln TYPE vbak-vbeln,
         kunnr TYPE vbak-kunnr,
         erdat TYPE vbak-erdat,
         ernam TYPE vbak-ernam,
         netwr TYPE vbak-netwr,
       END OF ty_vbak.

TYPES: BEGIN OF ty_vbap,
         vbeln TYPE vbap-vbeln,
         posnr TYPE vbap-posnr,
         matnr TYPE vbap-matnr,
         kwmeng TYPE vbap-kwmeng,
         netpr TYPE vbap-netpr,
       END OF ty_vbap.

TYPES: BEGIN OF ty_kna1,
         kunnr TYPE kna1-kunnr,
         name1 TYPE kna1-name1,
         land1 TYPE kna1-land1,
         regio TYPE kna1-regio,
       END OF ty_kna1.

TYPES: BEGIN OF ty_knvv,
         kunnr TYPE knvv-kunnr,
         vkorg TYPE knvv-vkorg,
         vtweg TYPE knvv-vtweg,
       END OF ty_knvv.

TYPES: BEGIN OF ty_mara,
         matnr TYPE mara-matnr,
         matkl TYPE mara-matkl,
         meins TYPE mara-meins,
       END OF ty_mara.

TYPES: BEGIN OF ty_mvke,
         matnr TYPE mvke-matnr,
         vkorg TYPE mvke-vkorg,
         vtweg TYPE mvke-vtweg,
       END OF ty_mvke.

TYPES: BEGIN OF ty_vbrk,
         vbeln TYPE vbrk-vbeln,
         fkdat TYPE vbrk-fkdat,
         netwr TYPE vbrk-netwr,
       END OF ty_vbrk.

TYPES: BEGIN OF ty_vbrp,
         vbeln TYPE vbrp-vbeln,
         posnr TYPE vbrp-posnr,
         matnr TYPE vbrp-matnr,
         kwmeng TYPE vbrp-kwmeng,
       END OF ty_vbrp.

* Define Table Types
TYPES: ty_vbak_tab TYPE TABLE OF ty_vbak,
       ty_vbap_tab TYPE TABLE OF ty_vbap,
       ty_kna1_tab TYPE TABLE OF ty_kna1,
       ty_knvv_tab TYPE TABLE OF ty_knvv,
       ty_mara_tab TYPE TABLE OF ty_mara,
       ty_mvke_tab TYPE TABLE OF ty_mvke,
       ty_vbrk_tab TYPE TABLE OF ty_vbrk,
       ty_vbrp_tab TYPE TABLE OF ty_vbrp.

* Selection Screen
PARAMETERS: p_vbeln TYPE vbak-vbeln.

* Start of Selection
START-OF-SELECTION.

  " Fetch Sales Order Header
  DATA lt_vbak TYPE ty_vbak_tab.
  SELECT vbeln, kunnr, erdat, ernam, netwr
    FROM vbak
    INTO TABLE lt_vbak
    WHERE vbeln = p_vbeln.

  IF lt_vbak IS INITIAL.
    MESSAGE 'Sales Order not found!' TYPE 'E'.
  ENDIF.

  " Fetch Sales Order Items
  DATA lt_vbap TYPE ty_vbap_tab.
  SELECT vbeln, posnr, matnr, kwmeng, netpr
    FROM vbap
    INTO TABLE lt_vbap
    FOR ALL ENTRIES IN lt_vbak
    WHERE vbeln = lt_vbak-vbeln.

  " Fetch Customer Data
  DATA lt_kna1 TYPE ty_kna1_tab.
  SELECT kunnr, name1, land1, regio
    FROM kna1
    INTO TABLE lt_kna1
    FOR ALL ENTRIES IN lt_vbak
    WHERE kunnr = lt_vbak-kunnr.

  " Fetch Customer Sales Data
  DATA lt_knvv TYPE ty_knvv_tab.
  SELECT kunnr, vkorg, vtweg
    FROM knvv
    INTO TABLE lt_knvv
    FOR ALL ENTRIES IN lt_vbak
    WHERE kunnr = lt_vbak-kunnr.

  " Fetch Material Data
  DATA lt_mara TYPE ty_mara_tab.
  SELECT matnr, matkl, meins
    FROM mara
    INTO TABLE lt_mara
    FOR ALL ENTRIES IN lt_vbap
    WHERE matnr = lt_vbap-matnr.

  " Fetch Material Sales Data
  DATA lt_mvke TYPE ty_mvke_tab.
  SELECT matnr, vkorg, vtweg
    FROM mvke
    INTO TABLE lt_mvke
    FOR ALL ENTRIES IN lt_vbap
    WHERE matnr = lt_vbap-matnr.

  " Fetch Billing Header Data
  DATA lt_vbrk TYPE ty_vbrk_tab.
  SELECT vbeln, fkdat, netwr
    FROM vbrk
    INTO TABLE lt_vbrk
    WHERE vbeln = p_vbeln.

  " Fetch Billing Item Data
  DATA lt_vbrp TYPE ty_vbrp_tab.
  SELECT vbeln, posnr, matnr, kwmeng
    FROM vbrp
    INTO TABLE lt_vbrp
    WHERE vbeln = p_vbeln.

  " Display Data in ALV
  TRY.
      DATA lo_alv TYPE REF TO cl_salv_table.
      cl_salv_table=>factory( IMPORTING r_salv_table = lo_alv CHANGING t_table = lt_vbak ).
      lo_alv->display( ).
  CATCH cx_salv_msg.
      MESSAGE 'Error displaying ALV' TYPE 'E'.
  ENDTRY.
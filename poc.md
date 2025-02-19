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
  lt_mvke TYPE TABLE OF ty_mvke.

FIELD-SYMBOLS: 
  <fs_kna1> TYPE ty_kna1,
  <fs_knvv> TYPE ty_knvv,
  <fs_mara> TYPE ty_mara,
  <fs_mvke> TYPE ty_mvke.

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

" Loop through customers
LOOP AT lt_knvv ASSIGNING <fs_knvv>.
  " Find a valid material
  LOOP AT lt_mvke ASSIGNING <fs_mvke> 
    WHERE vkorg = <fs_knvv>-vkorg 
    AND vtweg = <fs_knvv>-vtweg.

    WRITE: / 'Creating order for customer:', <fs_knvv>-kunnr,
            'Material:', <fs_mvke>-matnr,
            'Sales Org:', <fs_knvv>-vkorg,
            'Dist Channel:', <fs_knvv>-vtweg.
    
    EXIT. " Stop after first valid material
  ENDLOOP.
ENDLOOP.
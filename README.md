import React, { useState, useEffect, MouseEvent } from 'react';
import { MenuItem, Autocomplete, TextField, Button, Box, Typography, Grid, IconButton, Menu, CircularProgress } from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
import { useTranslation } from 'react-i18next';
import { useJobContext } from '../../../context/jobContext';
import { TOAST_TYPE } from '../../../utils/enums';
import PreviewQuery from '../../Common/previewQueryModal';
import { getScreenSize } from '../../../utils';
import { AppContextType, useAppContext } from '../../../context/appContext';
import { fetchOperationTagList, fetchOperationTagValueList } from '../services';
import { JobContextType } from '../interface/job.interface';
import JobDataTable from '../JobDataTable';
import { GridColDef } from '@mui/x-data-grid';

const OperationConfiguration: React.FC = () => {
  const { t } = useTranslation();
  const jobCtx: JobContextType = useJobContext();
  const appContext: AppContextType = useAppContext();
  const [operationTagList, setOperationTagList] = useState<string[]>([]);
  const [operationTagValueList, setOperationTagValueList] = useState<string[]>([]);
  const [selectedAutocompleteValue, setSelectedAutocompleteValue] = useState('');
  const [selectedTagValue, setSelectedTagValue] = useState('');
  const [rows, setRows] = useState<any[]>(jobCtx?.jobData?.operationConfigRows || []);
  const [anchorEl, setAnchorEl] = useState<null | HTMLElement>(null);
  const [selectedRowId, setSelectedRowId] = useState<number | null>(null);
  const [showQuery, setShowQuery] = useState<boolean>(false);
  const [fetchingTagValues, setFetchingTagValues] = useState<boolean>(false);
  const screenSize = getScreenSize();
  const [validationErrors, setValidationErrors] = useState<string[]>(new Array(rows.length).fill(''));

  const columns: GridColDef[] = [
    {
      field: t('operationNameField'),
      headerName: t('operationNameGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell'
    },
    {
      field: t('operationTagField'),
      headerName: t('operationTagGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell'
    },
    {
      field: t('operationTagValueField'),
      headerName: t('operationTagValueGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationTagValueColumn(params)
    },
    {
      field: t('operationSequenceField'),
      headerName: t('operationSequenceGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationSequenceColumn(params)
    },
    {
      field: t('operationTypeField'),
      headerName: t('operationTypeGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationTypeColumn(params)
    },
    {
      field: t('operationActionField'),
      headerName: t('operationActionGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 0.5,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderActionColumn(params)
    }
  ];

  /**
   * This useEffect update the autoComplete field , set the rows for datagrid and update jobCtx?.jobData
   * based on the the jobType selected.
   * also call the API for fetching the Operation Tags
   * @return {*}
   */
  useEffect(() => {
    setSelectedAutocompleteValue('');
    setRows([]);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: [] });

    if (jobCtx?.jobData?.jobType === t('snowflakeTransLabel') || jobCtx?.jobData?.jobType === t('snowflakeIngestionLabel')) {
      setSelectedAutocompleteValue(t('queryValue'));
    } else {
      handleFetchOperationTagList();
    }
  }, [jobCtx?.jobData?.jobType]);

  const handleFetchOperationTagList = () => {
    fetchOperationTagList()
      .then((arrayOfTagsList: string[]) => {
        setOperationTagList(arrayOfTagsList);
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
      });
  };

  const handleOperationTagValueOptionsChange = (searchValue?: string) => {
    setFetchingTagValues(true);
    fetchOperationTagValueList(searchValue, jobCtx?.jobData?.jobType)
      .then((arrayofTagValueList: string[]) => {
        setOperationTagValueList(arrayofTagValueList);
        setFetchingTagValues(false);
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
        setFetchingTagValues(false);
      });
  };

  /**
   * this method adds a row in datagrid component
   *
   * @return {*}
   */
  const handleAddOperation = () => {
    if (!selectedAutocompleteValue) {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.WARNING,
        toastMsg: t('selValidOpToastMsg')
      });
      return;
    }
    const newRows = [...rows];
    let newRow;
    if (jobCtx?.jobData?.jobType === t('snowflakeTransLabel') || jobCtx?.jobData?.jobType === t('snowflakeIngestionLabel')) {
      newRow = {
        id: newRows.length > 0 ? newRows[newRows.length - 1].id + 1 : 1,
        operationName: t('executeQueryLabel'),
        operationTag: selectedAutocompleteValue,
        operationTagValue: '',
        operationSequence: '', // Start with empty sequence
        action: ''
      };
    } else if (jobCtx?.jobData?.jobType === t('dbIngestionLabel')) {
      newRow = {
        id: newRows.length > 0 ? newRows[newRows.length - 1].id + 1 : 1,
        operationName: t('queryIngestionLabel'),
        operationTag: selectedAutocompleteValue,
        operationTagValue: '',
        operationSequence: '1', // Start with empty sequence
        action: ''
      };
    }
    newRows?.push(newRow);
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    setValidationErrors([...validationErrors, '']);
  };
  /**
   * this will conditionally render the operation tag value field
   * based on the job type selected
   * @param {*} params
   * @return {*}
   */
  const renderOperationTagValueColumn = (params?: any) => {
    const row = rows?.find((row) => row.id === params?.id);
    const error = validationErrors[params?.rowIndex];
    return row?.operationTag !== t('queryValue') && jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? (
      <TextField
        size="small"
        sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
        error={Boolean(error)}
        helperText={error || ''}
        FormHelperTextProps={{
          sx: { xs: { position: 'absolute', bottom: '-0.5vw' }, md: { position: 'absolute', bottom: '-1.2vw' } }
        }}
      />
    ) : (
      <Autocomplete
        ListboxProps={{ sx: { width: 'fit-content' } }}
        size="medium"
        className="tw-w-[20vw] tw-mt-5"
        options={operationTagValueList || []}
        value={selectedTagValue || ''}
        isOptionEqualToValue={(option?, value?) => option === value}
        onInputChange={(e?, searchValue?) => {
          handleOperationTagValueOptionsChange(searchValue);
        }}
        onChange={(e, selectedTagValue) => setSelectedTagValue(selectedTagValue as string)}
        renderInput={(params?) => (
          <TextField
            {...params}
            InputProps={{
              ...params?.InputProps,
              endAdornment: (
                <>
                  {fetchingTagValues && <CircularProgress size={20} />}
                  {params?.InputProps?.endAdornment}
                </>
              )
            }}
          />
        )}
      />
    );
  };

  /**
   * will render the operation sequence in the datagrid
   *
   * @param {*} params
   * @return {*}
   */
  const renderOperationSequenceColumn = (params?: any) => {
    const error = validationErrors[params?.rowIndex];
    return jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? (
      <TextField
        size="small"
        value={params?.value || ''}
        sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
        error={Boolean(error)}
        helperText={error || ''}
        FormHelperTextProps={{
          sx: { xs: { position: 'absolute', bottom: '-0.5vw' }, md: { position: 'absolute', bottom: '-1.2vw' } }
        }}
      />
    ) : (
      <TextField
        value={params?.value || ''}
        onKeyUp={(e?) => {
          if (!/[0-9]/.test(e?.key)) {
            e?.preventDefault();
          }
        }}
        type="text"
        inputProps={{ pattern: '[0-9]*' }}
        sx={{ '& .MuiInputBase-root': { height: '3rem' } }}
        className="tw-w-[15vw]"
        onChange={(event?) => handleSequenceChangeColumn(params?.id, event?.target?.value)}
        error={Boolean(error)}
        helperText={error || ''}
        FormHelperTextProps={{
          sx: { xs: { position: 'absolute', bottom: '-0.5vw' }, md: { position: 'absolute', bottom: '-1.2vw' } }
        }}
      />
    );
  };

  /**
   * if sequence number changes, then the rows data and jobCtx?.jobData will be updated
   *
   * @param {(string | number)} [id]
   * @param {*} [value]
   * @return {*}
   */
  const handleSequenceChangeColumn = (id?: string | number, value?: any) => {
    const regex = /^[0-9]+$/;
    if (!regex.test(value)) {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.ERROR,
        toastMsg: t('sequenceTypeError')
      });
      return;
    }

    const numericValue = Number(value);

    // Determine the maximum sequence number in current rows
    const maxAllowedValue = rows?.length > 0 ? Math.max(...rows?.map((row?) => row?.operationSequence)) : 0;

    // Check if the new value is already used in other rows
    if (rows.some((row) => row.id !== id && row.operationSequence === numericValue)) {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.ERROR,
        toastMsg: t('sequenceOrderError')
      });
      // console.error('This sequence number is already used. Please enter a unique sequence number.');
      return;
    }

    // Ensure the new sequence number follows the last one sequentially
    if (numericValue <= maxAllowedValue + 1) {
      setRows((prevRows) => prevRows.map((row) => (row.id === id ? { ...row, operationSequence: numericValue } : row)));
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: rows });
    } else {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.ERROR,
        toastMsg: t('sequenceOrderError')
      });
    }
  };

  /**
   * Rendering the Operation Type Column
   *
   * @param {*} [params]
   * @return {*}
   */
  const renderOperationTypeColumn = (params?: any) => {
    return (
      <TextField
        value={params?.value}
        onChange={(event?) => handleOperationTypeChange(params?.id, event?.target?.value)}
        sx={{ '& .MuiInputBase-root': { height: '3rem' } }}
        className="tw-w-[15vw]"
      />
    );
  };

  /**
   * Method to handle Operation Type Change
   *
   * @param {(string | number)} [id]
   * @param {(string | number)} [value]
   */
  const handleOperationTypeChange = (id?: string | number, value?: string | number) => {
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, renderOperationTypeColumn: value };
      }
      return row;
    });
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
  };

  /**
   * This renders the Action field in the datagrid
   *
   * @param {*} params
   * @return {*}
   */
  const renderActionColumn = (params?: any) => {
    return (
      <>
        <IconButton onClick={(event?) => handleActionMenuOpen(event, params?.id)}>
          <MoreVertIcon />
        </IconButton>
        <Menu anchorEl={anchorEl} open={Boolean(anchorEl) && selectedRowId === params?.id} onClose={handleActionMenuClose}>
          {rows?.find((row) => row.id === params.id)?.operationTag === t('queryValue') && (
            <MenuItem onClick={() => handleActionColumnItemClick('preview')}>{t('prevQueryLabel')}</MenuItem>
          )}
          <MenuItem onClick={() => handleActionColumnItemClick('delete')}>{t('delRowLabel')}</MenuItem>
        </Menu>
      </>
    );
  };

  /**
   * handles the Action menu for Datagrid
   *
   * @param {MouseEvent<HTMLElement>} event
   * @param {number} rowId
   */
  const handleActionMenuOpen = (event?: MouseEvent<HTMLElement>, rowId?: number) => {
    setAnchorEl(event?.currentTarget);
    setSelectedRowId(rowId);
  };

  /**
   * closes the menu
   *
   */
  const handleActionMenuClose = () => {
    setAnchorEl(null);
    setSelectedRowId(null);
  };

  /**
   * handles the click function on each option for menu item
   *
   * @param {string} action
   */
  const handleActionColumnItemClick = (action?: 'preview' | 'delete') => {
    if (action === 'delete') {
      const newRows = rows?.filter((row?) => row?.id !== selectedRowId);
      const reSequencedRows = reSequenceRowsOnDeletion(newRows);

      setRows(reSequencedRows);
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: reSequencedRows });
      setValidationErrors(validationErrors.filter((_, index) => index !== selectedRowId));
    }
    if (action === 'preview') {
      setShowQuery(true);
    }
    handleActionMenuClose();
  };

  /**
   * Resequencing method on deletion of rows
   *
   * @param {any[]} [rows]
   * @return {*}
   */
  const reSequenceRowsOnDeletion = (rows?: any[]) => {
    return rows?.map((row?, index?) => ({
      ...row,
      operationSequence: index + 1
    }));
  };

  const handleSave = () => {
    const errors: string[] = [];

    // Example validation logic
    rows.forEach((row, index) => {
      console.log('row', row);
      if (!row.operationTagValue) {
        errors[index] = 'Operation Tag is required.';
        console.log('error', errors[index]);
      }
      if (!row.operationSequence) {
        errors[index] = 'Operation Sequence is required.';
        console.log('error', errors[index]);
      }
      // Add more validations as per your fields' requirements
    });

    // Update validation errors state
    setValidationErrors(errors);

    // If there are no errors, proceed with saving
    if (errors.length === 0) {
      // Perform save operation here
      // Example: call an API or update state
      console.log('Data saved successfully.');
    }
  };

  console.log(validationErrors);
  return (
    <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue">
      <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
        {t('jobOperationConfigLabel')}
      </Typography>
      <Box padding={2}>
        <Grid container spacing={2} alignItems="center">
          <Grid item xs={12} sm={2} md={2} marginTop="1vh">
            <Typography sx={{ fontSize: 15, fontWeight: 'bold' }}>
              {t('searchOperationTagLabel')} <span style={{ color: 'red' }}> *</span>
            </Typography>
          </Grid>
          {jobCtx?.jobData?.jobType === t('snowflakeTransLabel') || jobCtx?.jobData?.jobType === t('snowflakeIngestionLabel') ? (
            <Grid item xs={12} sm={8} md={7}>
              <TextField
                value={t('queryValue')}
                sx={{ '& .MuiInputBase-root': { height: '3.25rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
                disabled
              />
            </Grid>
          ) : (
            <Grid item xs={12} sm={8} md={7}>
              <Autocomplete
                sx={{ width: { xs: '70vw', sm: '30vw', md: '30vw' } }}
                options={operationTagList || []}
                value={selectedAutocompleteValue || ''}
                isOptionEqualToValue={(option?, value?) => option === value}
                disabled={!jobCtx?.jobData?.jobType}
                onChange={(event?, newValue?) => setSelectedAutocompleteValue(newValue as string)}
                renderInput={(params?) => <TextField {...params} placeholder={t('searchOperationTagLabel')} fullWidth />}
              />
            </Grid>
          )}
          <Grid item xs={12} sm={12} md={3} container justifyContent={{ xs: 'flex-start', sm: 'flex-end' }}>
            <Button variant="contained" onClick={handleAddOperation} disabled={!jobCtx?.jobData?.jobType}>
              {t('addOperationBtnLabel')}
            </Button>
          </Grid>
        </Grid>

        <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', width: '100%', overflowX: 'auto', mt: 5 }}>
          <Box sx={{ width: { xs: '100vw', md: '100%' }, maxWidth: { xs: '100vw', md: '100%' }, padding: { xs: '4vw', md: 0 } }}>
            <JobDataTable rows={rows} columns={columns} />
          </Box>
        </Box>
      </Box>
      <Button variant="contained" onClick={handleSave} disabled={!jobCtx?.jobData?.jobType}>
        {t('Save')}
      </Button>
      {showQuery && <PreviewQuery showPreview={showQuery} setPreviewQuery={() => setShowQuery(false)} />}
    </Box>
  );
};

export default OperationConfiguration;

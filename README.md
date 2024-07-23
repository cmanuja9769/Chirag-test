import React, { useState, useEffect, MouseEvent, useRef } from 'react';
import { MenuItem, Autocomplete, TextField, Button, Box, Typography, Grid, IconButton, Menu, CircularProgress } from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
import { useTranslation } from 'react-i18next';
import { useJobContext } from '../../../context/jobContext';
import { TOAST_TYPE } from '../../../utils/enums';
import { getScreenSize } from '../../../utils';
import { AppContextType, useAppContext } from '../../../context/appContext';
import { fetchOperationTagList, fetchOperationTagValueList } from '../services';
import { JobContextType } from '../interface/job.interface';
import { DataGrid, GridColDef } from '@mui/x-data-grid';
import { Formik, Form, Field } from 'formik';
import EmptyDatagridComponent from '../../Common/emptyDatagridComponent';
import * as Yup from 'yup';

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
  const [fetchingTagValues, setFetchingTagValues] = useState<boolean>(false);
  const screenSize = getScreenSize();
  const formikRef = useRef<any>(null);

  const columns: GridColDef[] = [
    {
      field: 'operationName',
      headerName: t('operationNameGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell'
    },
    {
      field: 'operationTag',
      headerName: t('operationTagGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell'
    },
    {
      field: 'operationTagValue',
      headerName: t('operationTagValueGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationTagValueColumn(params)
    },
    {
      field: 'operationSequence',
      headerName: t('operationSequenceGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationSequenceColumn(params)
    },
    {
      field: 'operationType',
      headerName: t('operationTypeGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderOperationTypeColumn(params)
    },
    {
      field: 'action',
      headerName: t('operationActionGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 0.5,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params) => renderActionColumn(params)
    }
  ];

  const validForm = async () => {
    if (formikRef?.current) {
      try {
        await formikRef?.current?.submitForm();
      } catch (error) {}
    }
  };

  useEffect(() => {
    if (jobCtx?.isNextButtonClicked) {
      validForm();
      jobCtx?.handleNextClick(false);
    }
  }, [jobCtx?.isNextButtonClicked]);

  useEffect(() => {
    setSelectedAutocompleteValue('');
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

  const handleAddOperation = () => {
    if (!selectedAutocompleteValue) {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.WARNING,
        toastMsg: t('selValidOpToastMsg')
      });
      return;
    }
    const newRow = {
      id: rows.length > 0 ? rows[rows.length - 1].id + 1 : 1,
      operationName:
        jobCtx?.jobData?.jobType === t('snowflakeTransLabel') || jobCtx?.jobData?.jobType === t('snowflakeIngestionLabel')
          ? t('executeQueryLabel')
          : t('queryIngestionLabel'),
      operationTag: selectedAutocompleteValue,
      operationTagValue: '',
      operationSequence: '',
      operationType: '',
      action: ''
    };
    setRows((prevRows) => [...prevRows, newRow]);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: [...jobCtx?.jobData?.operationConfigRows, newRow] });
  };

  const renderOperationTagValueColumn = (params) => {
    const row = rows.find((row) => row.id === params.id);
    return (
      <Formik
        innerRef={formikRef}
        initialValues={{
          operationTagValue: row?.operationTagValue || ''
        }}
        validationSchema={Yup.object({
          operationTagValue:
            row?.operationTag !== t('queryValue') && jobCtx?.jobData?.jobType === t('dbIngestionLabel')
              ? Yup.string().required('Operation Tag Value is required.')
              : Yup.string()
        })}
        onSubmit={() => {
          // Handle form submission for individual row
        }}
      >
        {({ errors, touched, handleChange }) => (
          <Form>
            {row?.operationTag !== t('queryValue') && jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? (
              <Field
                as={TextField}
                name="operationTagValue"
                size="small"
                sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
                error={touched.operationTagValue && Boolean(errors.operationTagValue)}
                helperText={touched.operationTagValue && errors.operationTagValue}
                onChange={(e) => {
                  handleChange(e);
                  setRows((prevRows) => prevRows.map((row) => (row.id === params.id ? { ...row, operationTagValue: e.target.value } : row)));
                }}
              />
            ) : (
              <Autocomplete
                ListboxProps={{ sx: { width: 'fit-content' } }}
                size="medium"
                className="tw-w-[20vw] tw-mt-5"
                options={operationTagValueList || []}
                value={selectedTagValue || ''}
                isOptionEqualToValue={(option, value) => option === value}
                onInputChange={(e, searchValue) => {
                  handleOperationTagValueOptionsChange(searchValue);
                }}
                onChange={(e, newValue) => {
                  setSelectedTagValue(newValue || '');
                  setRows((prevRows) => prevRows.map((row) => (row.id === params.id ? { ...row, operationTagValue: newValue || '' } : row)));
                }}
                loading={fetchingTagValues}
                renderInput={(params) => (
                  <TextField
                    {...params}
                    InputProps={{
                      ...params.InputProps,
                      endAdornment: (
                        <>
                          {fetchingTagValues ? <CircularProgress color="inherit" size={20} /> : null}
                          {params.InputProps.endAdornment}
                        </>
                      )
                    }}
                  />
                )}
              />
            )}
          </Form>
        )}
      </Formik>
    );
  };

  const renderOperationSequenceColumn = (params) => {
    const row = rows.find((row) => row.id === params.id);
    return (
      <Formik
        innerRef={formikRef}
        initialValues={{
          operationSequence: row?.operationSequence || ''
        }}
        validationSchema={Yup.object({
          operationSequence: Yup.string().required('Operation Sequence is required.')
        })}
        onSubmit={() => {
          // Handle form submission for individual row
        }}
      >
        {({ errors, touched }) => (
          <Form>
            <Field
              as={TextField}
              name="operationSequence"
              size="small"
              sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
              error={touched.operationSequence && Boolean(errors.operationSequence)}
              helperText={touched.operationSequence && errors.operationSequence}
              onChange={(event?) => {
                handleSequenceChangeColumn(params?.id, event?.target?.value);
              }}
            />
          </Form>
        )}
      </Formik>
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

  const renderOperationTypeColumn = (params) => {
    const row = rows.find((row) => row.id === params.id);
    return (
      <Formik
        initialValues={{
          operationType: row?.operationType || ''
        }}
        validationSchema={Yup.object({
          operationType: Yup.string().required('Operation Type is required.')
        })}
        onSubmit={() => {
          // Handle form submission for individual row
        }}
      >
        {({ errors, touched }) => (
          <Form>
            <Field
              as={TextField}
              name="operationType"
              size="small"
              sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
              error={touched.operationType && Boolean(errors.operationType)}
              helperText={touched.operationType && errors.operationType}
              onChange={(event?) => {
                handleOperationTypeChange(params?.id, event?.target?.value);
              }}
            />
          </Form>
        )}
      </Formik>
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
    }
    if (action === 'preview') {
      // setShowQuery(true);
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
            <DataGrid
              autoHeight
              rows={rows}
              columns={columns}
              pageSizeOptions={[10]}
              slots={{ noRowsOverlay: EmptyDatagridComponent }}
              disableRowSelectionOnClick
              disableColumnFilter
              disableColumnSelector
              disableDensitySelector
              disableColumnMenu
              disableColumnResize
              rowHeight={100}
              className="tw-min-w-[30vw]"
              sx={{
                '--DataGrid-overlayHeight': '50vh',
                '& .custom-header': { backgroundColor: '#F2F4F8', color: 'black' },
                '& .custom-header-sequence': { backgroundColor: '#F2F4F8', color: 'black', paddingLeft: '3vw' },
                '& .custom-cell': {
                  backgroundColor: 'white',
                  padding: '0.625rem',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'left',
                  minHeight: '5vh'
                },
                '& .custom-cell-sequence': {
                  backgroundColor: 'white',
                  padding: '0.625rem',
                  paddingLeft: '3vw',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'left',
                  minHeight: '5vh'
                },
                '& .MuiDataGrid-cell:focus-within': {
                  outline: 'none !important'
                }
              }}
            />
          </Box>
        </Box>
      </Box>
    </Box>
  );
};

export default OperationConfiguration;

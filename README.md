import React, { useState, useEffect, MouseEvent } from 'react';
import { MenuItem, Autocomplete, TextField, Button, Box, Typography, Grid, IconButton, Menu } from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
import { DataGrid, GridCellParams } from '@mui/x-data-grid';
import EmptyDatagridComponent from '../../Common/emptyDatagridComponent';
import { useTranslation } from 'react-i18next';
import { useJobContext } from '../../../context/jobContext';
import { TOAST_TYPE } from '../../../utils/enums';
import PreviewQuery from '../../Common/previewQueryModal';
import { AppContextType, useAppContext } from '../../../context/appContext';
import { fetchOperationTagValueList } from '../services';
import { JobContextType } from '../interface/job.interface';
import * as Yup from 'yup';
import { Form, Formik, useFormik } from 'formik';
import { appConstant } from '../../../utils/constants/appConstants';
import { fetchConnectionList, fetchObjectList, fetchSchemaList } from '../../queries/services';
import JobTextField from '../jobDatagridCellComponents/jobTextField';
import { operationConfigurationColumns } from '../constants/jobConstants';
import JobAutocomplete from '../jobDatagridCellComponents/jobAutocomplete';
import colors from '../../../styles/colorPalette';

const OperationConfiguration: React.FC = () => {
  const { t } = useTranslation();
  const jobCtx: JobContextType = useJobContext();
  const appContext: AppContextType = useAppContext();
  const [operationTagValueList, setOperationTagValueList] = useState<{ id: string; name: string }[]>([]);
  const [selectedAutocompleteValue, setSelectedAutocompleteValue] = useState('');
  const [rows, setRows] = useState<any[]>([]);
  const [anchorEl, setAnchorEl] = useState<null | HTMLElement>(null);
  const [selectedRowId, setSelectedRowId] = useState<number | null>(null);
  const [showQuery, setShowQuery] = useState<boolean>(false);
  const [loadingTagValues, setLoadingTagValues] = useState({});
  const [connectionOptions, setConnectionOptions] = useState<{ CONNECTION_ID: string; CONNECTION_NAME: string }[]>([]);
  const [schemaOptions, setSchemaOptions] = useState<{ SCHEMA_NAME: string; ENTITY_NAME: string }[]>([]);

  let options: { id: string; name: string }[] = [];
  let apiCall: any;
  let apiParams: any[];

  const operationConfigColumns = operationConfigurationColumns?.map((column?) => {
    switch (column?.field) {
      case appConstant?.jobOperationCellNames?.operationTag:
        return {
          ...column,
          renderCell: (params?: GridCellParams) => renderOperationTagColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationTagValue:
        return {
          ...column,
          renderCell: (params?: GridCellParams) => renderOperationTagValueColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationSequence:
        return {
          ...column,
          renderCell: (params?: GridCellParams) => renderOperationSequenceColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationType:
        return {
          ...column,
          renderCell: (params?: GridCellParams) => renderOperationTypeColumn(params)
        };
      case appConstant?.jobOperationCellNames?.action:
        return {
          ...column,
          renderCell: (params?: GridCellParams) => renderActionColumn(params)
        };
      default:
        return column;
    }
  });

  useEffect(() => {
    if (!jobCtx?.isEditJob) {
      setSelectedAutocompleteValue('');
      setRows([]);
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: [] });
      formik?.setFieldValue('rows', []);
      jobCtx?.updateOperationTagList([]);
    }
  }, [jobCtx?.jobData?.jobType]);

  useEffect(() => {
    if (!jobCtx?.isEditJob) {
      const mandatoryOperationTags = jobCtx?.operationTagList
        .filter((item?) => item?.ENTITY_KEYS?.operationType === appConstant?.mandatory)
        .map((item?) => item?.ENTITY_NAME);
      mandatoryOperationTags?.forEach((mandatoryTag?) => {
        handleAddOperation(mandatoryTag);
      });
    }
  }, [jobCtx?.operationTagList]);

  useEffect(() => {
    if (jobCtx?.isNextButtonClicked) {
      handleSave()
        .then(() => {
          jobCtx?.updateIsFormValid(true);
        })
        .catch(() => {
          jobCtx?.updateIsFormValid(false);
        });
    }
  }, [jobCtx?.isNextButtonClicked]);

  useEffect(() => {
    if (jobCtx?.isEditJob) {
      const formattedRows = jobCtx?.jobData?.operationConfigRows?.map((row?, index?) => ({
        id: index + 1,
        ...row
      }));

      setRows(formattedRows);
      formik?.setFieldValue('rows', formattedRows);
    }
  }, [jobCtx?.jobData?.operationConfigRows, jobCtx?.isEditJob]);

  const handleSave = () => {
    return new Promise(async (resolve, reject) => {
      const errors = await formik?.validateForm();
      formik?.setTouched({
        rows: formik?.values?.rows?.map(() => {
          return { operationTag: true, operationTagValue: true, operationSequence: true, operationType: true };
        })
      });
      await formik?.handleSubmit();
      if (Object.keys(errors)?.length === 0 && jobCtx?.jobData?.operationConfigRows?.length !== 0) {
        return resolve(true);
      } else {
        return reject(false);
      }
    });
  };

  const validationSchema = Yup?.object()?.shape({
    rows: Yup?.array()?.of(
      Yup?.object()?.shape({
        operationTag: Yup?.string()?.required(),
        operationTagValue: Yup?.string()?.required(),
        operationSequence: Yup?.string()?.required(),
        operationType: Yup?.string()?.required()
      })
    )
  });

  const formik = useFormik({
    initialValues: { rows },
    validationSchema,
    onSubmit: (values?: { rows: Record<string, any>[] }) => {
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: values?.rows });
    }
  });

  /**
   * this method adds a row in datagrid component
   *
   * @return {*}
   */
  let newRows = [...rows];
  const handleAddOperation = (mandatoryTag?: string) => {
    const jobOperationName = jobCtx?.jobData?.jobType?.operationName;
    let newRow;
    newRow = {
      id: newRows?.length > 0 ? newRows[newRows?.length - 1]?.id + 1 : 1,
      operationName: jobOperationName === appConstant?.operationName?.executeQuery ? t('executeQuery') : t('queryIngestion'),
      operationTag: mandatoryTag ?? selectedAutocompleteValue,
      operationTagColumn: mandatoryTag ?? selectedAutocompleteValue,
      operationTagValue: '',
      operationTagValueGui: '',
      operationSequence: jobOperationName === appConstant?.operationName?.executeQuery ? '' : '1',
      operationType: '',
      action: ''
    };

    newRows?.push(newRow);
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', [...newRows, newRow]);
    setSelectedAutocompleteValue('');
  };

  const renderOperationTagColumn = (params?: any) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    return row?.operationTagColumn === t('createCustomTagLabel') ? (
      <JobTextField
        params={params}
        rows={rows}
        formik={formik}
        handleChange={handleOperationTagChange}
        cellName={appConstant?.jobOperationCellNames?.operationTag}
      />
    ) : (
      <span>{params?.value}</span>
    );
  };

  const handleOperationTagChange = (id?: number, value?: string | number) => {
    formik?.setFieldValue(`rows[${id - 1}].operationTag`, value);
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, operationTag: value };
      }
      return row;
    });
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', newRows);
  };

  /**
   * this will conditionally render the operation tag value field
   * based on the job type selected
   * @param {*} params
   * @return {*}
   */
  const renderOperationTagValueColumn = (params?: any) => {
    const row = rows?.find((row) => row?.id === params?.id);
    const isMandatoryTag = appConstant?.mandatoryTagsList?.includes(row?.operationTag);
    return !isMandatoryTag ? (
      <JobTextField
        params={params}
        rows={rows}
        formik={formik}
        handleChange={handleOperationTagValueChange}
        cellName={appConstant?.jobOperationCellNames?.operationTagValueGui}
      />
    ) : (
      <JobAutocomplete
        params={params}
        rows={rows}
        operationTagValueList={operationTagValueList}
        handleOperationTagValueChange={handleOperationTagValueChange}
        handleOperationTagValueOptionsChange={handleOperationTagValueOptionsChange}
        setOperationTagValueList={setOperationTagValueList}
        loadingTagValues={loadingTagValues[row.id]}
        handleFetchSchemaObjectChange={handleFetchSchemaObjectChange}
        value={formik?.values?.rows[params?.id - 1]?.operationTagValueGui}
      />
    );
  };

  const handleOperationTagValueOptionsChange = (searchValue?: string, params?: GridCellParams) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    setLoadingTagValues((prev?) => ({ ...prev, [params?.id]: true }));
    switch (row?.operationTag) {
      case appConstant?.jobOperationTagNames?.connectionId:
        apiCall = fetchConnectionList;
        apiParams = [searchValue];
        break;
      case appConstant?.jobOperationTagNames?.query:
        apiCall = fetchOperationTagValueList;
        apiParams = [searchValue, jobCtx?.jobData?.jobType];
        break;
      default:
        break;
    }
    apiCall(...apiParams)
      .then((response: { [key: string]: any }) => {
        // Map API responses to objects with id and name
        switch (row.operationTag) {
          case appConstant?.jobOperationTagNames?.connectionId:
            setConnectionOptions(response?.data);
            options = response?.data?.map((item?) => ({
              id: item?.CONNECTION_ID,
              name: item?.CONNECTION_NAME
            }));
            break;
          case appConstant?.jobOperationTagNames?.query:
            options = response?.data?.map((item?) => ({
              id: item?.RULE_ID,
              name: item?.RULE_KEY
            }));
            break;
          default:
            break;
        }
        setOperationTagValueList(options); // options is now { id: string; name: string }[]
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
      })
      .finally(() => {
        setLoadingTagValues((prev) => ({ ...prev, [params?.id]: false }));
      });
  };

  const handleFetchSchemaObjectChange = (params?: GridCellParams) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    const connectionId = connectionOptions?.find(
      (option?) =>
        option?.CONNECTION_NAME === rows?.find((r?) => r?.operationTag === appConstant?.jobOperationTagNames?.connectionId)?.operationTagValueGui
    );
    const schemaId = schemaOptions?.find(
      (option?) =>
        option?.ENTITY_NAME === rows?.find((r?) => r?.operationTag === appConstant?.jobOperationTagNames?.writeSchemaName)?.operationTagValueGui
    );
    switch (row?.operationTag) {
      case appConstant?.jobOperationTagNames?.writeSchemaName:
        if (!connectionId) {
          appContext?.updateToastAttributes({
            showToast: true,
            severity: TOAST_TYPE.WARNING,
            toastMsg: t('selectConnIdWarning')
          });
          return;
        }
        apiCall = fetchSchemaList;
        apiParams = [connectionId?.CONNECTION_ID];
        break;
      case appConstant?.jobOperationTagNames?.writeTableName:
        if (!schemaId || !connectionId) {
          appContext?.updateToastAttributes({
            showToast: true,
            severity: TOAST_TYPE.WARNING,
            toastMsg: t('selectSchemaIdWarning')
          });
          return;
        }
        apiCall = fetchObjectList;
        apiParams = [connectionId?.CONNECTION_ID, schemaId?.SCHEMA_NAME];
        break;
      default:
        break;
    }
    setLoadingTagValues((prev?) => ({ ...prev, [params?.id]: true }));
    apiCall(...apiParams)
      .then((response: { [key: string]: any }) => {
        // Map API responses to objects with id and name
        switch (row.operationTag) {
          case appConstant?.jobOperationTagNames?.writeSchemaName:
            setSchemaOptions(response?.data?.schema);
            options = response?.data?.schema?.map((item?) => ({
              id: item?.SCHEMA_NAME,
              name: item?.ENTITY_NAME
            }));
            break;
          case appConstant?.jobOperationTagNames?.writeTableName:
            options = response?.data?.objects?.map((item?) => ({
              id: item,
              name: item
            }));
            break;
          default:
            break;
        }
        setOperationTagValueList(options); // options is now { id: string; name: string }[]
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
      })
      .finally(() => {
        setLoadingTagValues((prev) => ({ ...prev, [params?.id]: false }));
      });
  };

  const handleOperationTagValueChange = (id?: number, tagValueGui?: string, tagValue?: number) => {
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, operationTagValue: tagValue, operationTagValueGui: tagValueGui };
      }
      return row;
    });
    formik?.setFieldValue(`rows[${id - 1}].operationTagValueGui`, tagValueGui);
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', newRows);
  };

  /**
   * will render the operation sequence in the datagrid
   *
   * @param {*} params
   * @return {*}
   */
  const renderOperationSequenceColumn = (params?: any) => {
    const isQueryIngestion = jobCtx?.jobData?.jobType?.operationName === appConstant?.operationName?.queryIngestion;
    return (
      <JobTextField
        params={params}
        rows={rows}
        formik={formik}
        isReadOnly={isQueryIngestion ? true : false}
        handleChange={isQueryIngestion ? () => {} : handleSequenceChangeColumn}
        inputProps={isQueryIngestion ? {} : { pattern: '[0-9]*' }}
        cellName={appConstant?.jobOperationCellNames?.operationSequence}
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
  const handleSequenceChangeColumn = (id?: number, value?: any) => {
    // Check if the input value is a valid number
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
    if (rows?.some((row?) => row?.id !== id && row?.operationSequence === numericValue)) {
      appContext?.updateToastAttributes({
        showToast: true,
        severity: TOAST_TYPE.ERROR,
        toastMsg: t('sequenceOrderError')
      });
      return;
    }
    // Ensure the new sequence number follows the last one sequentially
    if (numericValue <= maxAllowedValue + 1) {
      setRows((prevRows?) => prevRows?.map((row?) => (row?.id === id ? { ...row, operationSequence: numericValue } : row)));
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: rows });
      formik?.setFieldValue('rows', [...rows, rows]);
      formik?.setFieldValue(`rows[${id - 1}].operationSequence`, value);
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
      <JobTextField
        params={params}
        rows={rows}
        formik={formik}
        handleChange={handleOperationTypeChange}
        cellName={appConstant?.jobOperationCellNames?.operationType}
      />
    );
  };

  /**
   * Method to handle Operation Type Change
   *
   * @param {(string | number)} [id]
   * @param {(string | number)} [value]
   */
  const handleOperationTypeChange = (id?: number, value?: string | number) => {
    formik?.setFieldValue(`rows[${id - 1}].operationType`, value);
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, operationType: value };
      }
      return row;
    });
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', newRows);
  };

  /**
   * This renders the Action field in the datagrid
   *
   * @param {*} params
   * @return {*}
   */
  const renderActionColumn = (params?: any) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    return (
      <>
        <IconButton onClick={(event?) => handleActionMenuOpen(event, params?.id)}>
          <MoreVertIcon />
        </IconButton>
        <Menu anchorEl={anchorEl} open={Boolean(anchorEl) && selectedRowId === params?.id} onClose={handleActionMenuClose}>
          {appConstant?.mandatoryTagsList?.includes(row?.operationTag) && (
            <MenuItem
              disabled={row?.operationTagValue === '' || row?.operationTagValue === null}
              onClick={() => handleActionColumnItemClick('preview')}
            >
              {t('prevQueryLabel')}
            </MenuItem>
          )}
          {!appConstant?.mandatoryTagsList?.includes(row?.operationTag) && (
            <MenuItem onClick={() => handleActionColumnItemClick('delete')}>{t('delRowLabel')}</MenuItem>
          )}
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
      formik?.setFieldValue('rows', [...rows, reSequencedRows]);
      jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: reSequencedRows });
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

  return (
    <Formik initialValues={formik.initialValues} validationSchema={validationSchema} onSubmit={() => handleSave()}>
      <Form>
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
              <Grid item xs={12} sm={8} md={7}>
                <Autocomplete
                  sx={{ width: { xs: '70vw', sm: '30vw', md: '30vw' } }}
                  options={
                    jobCtx?.operationTagList
                      ?.filter((item?) => item?.ENTITY_KEYS?.operationType !== appConstant?.mandatory)
                      .map((item?) => item?.ENTITY_NAME) || []
                  }
                  value={selectedAutocompleteValue || ''}
                  isOptionEqualToValue={(option?, value?) => option === value}
                  disabled={!jobCtx?.jobData?.jobType}
                  onChange={(event?, newValue?) => setSelectedAutocompleteValue(newValue as string)}
                  renderInput={(params?) => <TextField {...params} placeholder={t('searchOperationTagLabel')} fullWidth />}
                />
              </Grid>
              <Grid item xs={12} sm={12} md={3} container justifyContent={{ xs: 'flex-start', sm: 'flex-end' }}>
                <Button variant="contained" onClick={() => handleAddOperation()} disabled={!selectedAutocompleteValue}>
                  {t('addOperationBtnLabel')}
                </Button>
              </Grid>
            </Grid>

            <Box sx={{ display: 'flex', justifyContent: 'center', alignItems: 'center', width: '100%', overflowX: 'auto', mt: 5 }}>
              <Box sx={{ width: { xs: '100vw', md: '100%' }, maxWidth: { xs: '100vw', md: '100%' }, padding: { xs: '4vw', md: 0 } }}>
                <DataGrid
                  autoHeight
                  rows={rows}
                  columns={operationConfigColumns}
                  pageSizeOptions={[10]}
                  slots={{ noRowsOverlay: EmptyDatagridComponent }}
                  disableRowSelectionOnClick
                  disableColumnFilter
                  disableColumnSelector
                  disableDensitySelector
                  disableColumnMenu
                  disableColumnResize
                  rowHeight={70}
                  className="tw-min-w-[30vw]"
                  sx={{
                    '--DataGrid-overlayHeight': '50vh',
                    '& .custom-header': { backgroundColor: colors?.dagHeader, color: colors?.black },
                    '& .custom-header-sequence': { backgroundColor: colors?.dagHeader, color: colors?.black, paddingLeft: '3vw' },
                    '& .custom-cell': {
                      backgroundColor: colors?.white,
                      padding: '0.625rem',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'left',
                      minHeight: '5vh'
                    },
                    '& .custom-cell-sequence': {
                      backgroundColor: colors?.white,
                      padding: '0.625rem',
                      paddingLeft: '3vw',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'left',
                      minHeight: '5vh'
                    },
                    '& .MuiDataGrid-cell:focus-within': {
                      outline: 'none'
                    }
                  }}
                />
              </Box>
            </Box>
          </Box>
          {showQuery && <PreviewQuery showPreview={showQuery} setPreviewQuery={() => setShowQuery(false)} />}
        </Box>
      </Form>
    </Formik>
  );
};

export default OperationConfiguration;

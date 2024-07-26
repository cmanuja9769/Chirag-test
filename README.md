import React, { useState, useEffect, MouseEvent } from 'react';
import { MenuItem, Autocomplete, TextField, Button, Box, Typography, Grid, IconButton, Menu } from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
import { DataGrid, GridColDef } from '@mui/x-data-grid';
import EmptyDatagridComponent from '../../Common/emptyDatagridComponent';
import { useTranslation } from 'react-i18next';
import { useJobContext } from '../../../context/jobContext';
import { TOAST_TYPE } from '../../../utils/enums';
import PreviewQuery from '../../Common/previewQueryModal';
import { getScreenSize } from '../../../utils';
import { AppContextType, useAppContext } from '../../../context/appContext';
import { fetchOperationTagValueList } from '../services';
import { JobContextType } from '../interface/job.interface';
import * as Yup from 'yup';
import { FormikErrors, useFormik } from 'formik';
import { identifyMandatoryTags } from '../../../utils/constants/appConstants';

const OperationConfiguration: React.FC = () => {
  const { t } = useTranslation();
  const jobCtx: JobContextType = useJobContext();
  const appContext: AppContextType = useAppContext();
  const [operationTagValueList, setOperationTagValueList] = useState<string[]>([]);
  const [selectedAutocompleteValue, setSelectedAutocompleteValue] = useState('');
  const [rows, setRows] = useState<any[]>([]);
  const [anchorEl, setAnchorEl] = useState<null | HTMLElement>(null);
  const [selectedRowId, setSelectedRowId] = useState<number | null>(null);
  const [showQuery, setShowQuery] = useState<boolean>(false);
  const screenSize = getScreenSize();

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
      cellClassName: 'custom-cell',
      renderCell: (params?) => renderOperationTagColumn(params)
    },
    {
      field: t('operationTagValueField'),
      headerName: t('operationTagValueGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params?) => renderOperationTagValueColumn(params)
    },
    {
      field: t('operationSequenceField'),
      headerName: t('operationSequenceGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params?) => renderOperationSequenceColumn(params)
    },
    {
      field: t('operationTypeField'),
      headerName: t('operationTypeGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 1,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params?) => renderOperationTypeColumn(params)
    },
    {
      field: t('operationActionField'),
      headerName: t('operationActionGUI'),
      width: screenSize?.isSmallScreen ? 150 : undefined,
      flex: screenSize?.isSmallScreen ? undefined : 0.5,
      minWidth: 50,
      headerClassName: 'custom-header',
      cellClassName: 'custom-cell',
      renderCell: (params?) => renderActionColumn(params)
    }
  ];

  useEffect(() => {
    setSelectedAutocompleteValue('');
    setRows([]);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: [] });
    formik?.setFieldValue('rows', []);
    jobCtx?.updateOperationTagList([]);
  }, [jobCtx?.jobData?.jobType]);

  useEffect(() => {
    const mandatoryOperationTags = jobCtx?.operationTagList
      .filter((item?) => item?.ENTITY_KEYS?.operationType === t('mandatoryOperationTag'))
      .map((item?) => item?.ENTITY_NAME);
    mandatoryOperationTags?.forEach((mandatoryTag?) => {
      handleAddOperation(mandatoryTag);
    });
  }, [jobCtx?.operationTagList]);

  useEffect(() => {
    if (jobCtx?.isNextButtonClicked) {
      handleSave()
        .then(() => {
          console.log('i am execting the correct form from opCOn');
          jobCtx?.updateIsFormInvalid(false);
        })
        .catch(() => {
          console.log('i am execting the invalid form opCOn');
          jobCtx?.updateIsFormInvalid(true);
        });
    }
  }, [jobCtx?.isNextButtonClicked]);

  const handleSave = () => {
    return new Promise(async (resolve, reject) => {
      formik?.validateForm()?.then((errors?: FormikErrors<{ rows?: any[] }>) => {
        formik?.setTouched({
          rows: formik?.values?.rows?.map(() => {
            return { operationTag: true, operationTagValue: true, operationSequence: true, operationType: true };
          })
        });
        if (Object.keys(errors)?.length === 0) {
          formik?.handleSubmit();
          return resolve(true);
        } else {
          return reject(true);
        }
      });
      console.log('jobData', jobCtx?.jobData);
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
    onSubmit: () => {}
  });

  const handleOperationTagValueOptionsChange = (searchValue?: string) => {
    fetchOperationTagValueList(searchValue, jobCtx?.jobData?.jobType)
      .then((arrayofTagValueList: string[]) => {
        setOperationTagValueList(arrayofTagValueList);
      })
      .catch(() => {
        appContext?.updateToastAttributes({
          showToast: true,
          severity: TOAST_TYPE.ERROR,
          toastMsg: t('apiError')
        });
      });
  };

  const handleOperationTagValueChange = (value?: string, id?: number) => {
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, operationTagValue: value };
      }
      return row;
    });
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', newRows);
  };

  /**
   * this method adds a row in datagrid component
   *
   * @return {*}
   */
  let newRows = [...rows];
  const handleAddOperation = (mandatoryTag?: string) => {
    let newRow;

    newRow = {
      id: newRows?.length > 0 ? newRows[newRows?.length - 1]?.id + 1 : 1,
      operationName: jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? t('queryIngestionLabel') : t('executeQueryLabel'),
      operationTag: mandatoryTag ?? selectedAutocompleteValue,
      operationTagColumn: mandatoryTag ?? selectedAutocompleteValue,
      operationTagValue: '',
      operationSequence: jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? '1' : '',
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
      <TextField
        size="small"
        sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
        value={formik?.values?.rows[params?.id - 1]?.operationTag || ''}
        onChange={(event?) => {
          handleOperationTagChange(params?.id, event?.target?.value);
        }}
        error={
          formik?.touched?.rows &&
          formik?.touched?.rows[params?.id - 1] &&
          formik?.errors?.rows &&
          (formik?.errors?.rows[params?.id - 1] as any)?.operationTag
        }
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
    const row = rows?.find((row?) => row?.id === params?.id);
    return !identifyMandatoryTags[row?.operationTag] ? (
      <TextField
        size="small"
        sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
        value={formik?.values?.rows[params?.id - 1]?.operationTagValue || ''}
        onChange={(e?) => {
          handleOperationTagValueChange(e?.target?.value, params?.id);
          formik?.setFieldValue(`rows[${params?.id - 1}].operationTagValue`, e?.target?.value);
        }}
        error={
          formik?.touched?.rows &&
          formik?.touched?.rows[params?.id - 1] &&
          formik?.errors?.rows &&
          (formik?.errors?.rows[params?.id - 1] as any)?.operationTagValue
        }
      />
    ) : (
      <Autocomplete
        ListboxProps={{ sx: { width: 'fit-content' } }}
        size="medium"
        className="tw-w-[20vw] tw-mt-5"
        options={operationTagValueList || []}
        value={formik?.values?.rows[params?.id - 1]?.operationTagValue || ''}
        isOptionEqualToValue={(option?, value?) => option === value}
        onInputChange={(e?, searchValue?) => {
          handleOperationTagValueOptionsChange(searchValue);
          formik?.setFieldValue(`rows[${params?.id - 1}].operationTagValue`, searchValue);
        }}
        onChange={(e, selectedTagValue) => {
          handleOperationTagValueChange(selectedTagValue, params?.id);
          formik?.setFieldValue(`rows[${params?.id - 1}].operationTagValue`, selectedTagValue);
        }}
        renderInput={(params?: any) => {
          return (
            <TextField
              {...params}
              error={
                formik?.touched?.rows &&
                formik?.touched?.rows[params?.id - 1] &&
                formik?.errors?.rows &&
                (formik?.errors?.rows[params?.id - 1] as any)?.operationTagValue
              }
            />
          );
        }}
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
    return jobCtx?.jobData?.jobType === t('dbIngestionLabel') ? (
      <TextField
        size="small"
        value={formik?.values?.rows[params?.id - 1]?.operationSequence || ''}
        sx={{ '& .MuiInputBase-root': { height: '3rem' }, 'width': { xs: '70vw', sm: '30vw', md: '30vw' } }}
        onChange={(event?) => {
          formik?.setFieldValue(`rows[${params?.id - 1}].operationSequence`, event?.target?.value);
        }}
        error={
          formik?.touched?.rows &&
          formik?.touched?.rows[params?.id - 1] &&
          formik?.errors?.rows &&
          (formik?.errors?.rows[params?.id - 1] as any)?.operationSequence
        }
        InputProps={{
          readOnly: true
        }}
      />
    ) : (
      <>
        <TextField
          value={formik?.values?.rows[params?.id - 1]?.operationSequence || ''}
          onKeyUp={(e?) => {
            if (!/[0-9]/.test(e?.key)) {
              e?.preventDefault();
            }
          }}
          type="text"
          inputProps={{ pattern: '[0-9]*' }}
          sx={{ '& .MuiInputBase-root': { height: '3rem' } }}
          className="tw-w-[15vw]"
          onChange={(event?) => {
            handleSequenceChangeColumn(params?.id, event?.target?.value);
          }}
          error={
            formik?.touched?.rows &&
            formik?.touched?.rows[params?.id - 1] &&
            formik?.errors?.rows &&
            (formik?.errors?.rows[params?.id - 1] as any)?.operationSequence
          }
        />
      </>
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
      formik?.setFieldValue(`rows[${id - 1}].operationSequence`, value);
      formik?.setFieldValue('rows', [...rows, rows]);
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
        value={formik?.values?.rows[params?.id - 1]?.operationType || ''}
        onChange={(event?) => {
          event.preventDefault();
          handleOperationTypeChange(params?.id, event?.target?.value);
        }}
        sx={{ '& .MuiInputBase-root': { height: '3rem' } }}
        className="tw-w-[15vw]"
        error={
          formik?.touched?.rows &&
          formik?.touched?.rows[params?.id - 1] &&
          formik?.errors?.rows &&
          (formik?.errors?.rows[params?.id - 1] as any)?.operationType
        }
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
          {identifyMandatoryTags[row?.operationTag] && (
            <MenuItem
              disabled={row.operationTagValue === '' || row.operationTagValue === null}
              onClick={() => handleActionColumnItemClick('preview')}
            >
              {t('prevQueryLabel')}
            </MenuItem>
          )}
          {!identifyMandatoryTags[row?.operationTag] && <MenuItem onClick={() => handleActionColumnItemClick('delete')}>{t('delRowLabel')}</MenuItem>}
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
                  ?.filter((item?) => item?.ENTITY_KEYS?.operationType !== t('mandatoryOperationTag'))
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
              columns={columns}
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
                  outline: 'none'
                }
              }}
            />
          </Box>
        </Box>
      </Box>
      {showQuery && <PreviewQuery showPreview={showQuery} setPreviewQuery={() => setShowQuery(false)} />}
    </Box>
  );
};

export default OperationConfiguration;

import React, { useState, useEffect, MouseEvent } from 'react';
import { MenuItem, Autocomplete, TextField, Button, Box, Typography, Grid, IconButton, Menu, Table, TableBody, TableCell, TableContainer, TableHead, TableRow, Paper } from '@mui/material';
import MoreVertIcon from '@mui/icons-material/MoreVert';
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
          renderCell: (params?: any) => renderOperationTagColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationTagValue:
        return {
          ...column,
          renderCell: (params?: any) => renderOperationTagValueColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationSequence:
        return {
          ...column,
          renderCell: (params?: any) => renderOperationSequenceColumn(params)
        };
      case appConstant?.jobOperationCellNames?.operationType:
        return {
          ...column,
          renderCell: (params?: any) => renderOperationTypeColumn(params)
        };
      case appConstant?.jobOperationCellNames?.action:
        return {
          ...column,
          renderCell: (params?: any) => renderActionColumn(params)
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
        handleChange={handleOperationTagValueChange}
        cellName={appConstant?.jobOperationCellNames?.operationTagValue}
      />
    );
  };

  const handleOperationTagValueChange = (id?: number, value?: string | number) => {
    formik?.setFieldValue(`rows[${id - 1}].operationTagValue`, value);
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

  const renderOperationSequenceColumn = (params?: any) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    return (
      <JobTextField
        params={params}
        rows={rows}
        formik={formik}
        handleChange={handleOperationSequenceChange}
        cellName={appConstant?.jobOperationCellNames?.operationSequence}
      />
    );
  };

  const handleOperationSequenceChange = (id?: number, value?: string | number) => {
    formik?.setFieldValue(`rows[${id - 1}].operationSequence`, value);
    const newRows = rows?.map((row?) => {
      if (row?.id === id) {
        return { ...row, operationSequence: value };
      }
      return row;
    });
    setRows(newRows);
    jobCtx?.updateJobData({ ...jobCtx?.jobData, operationConfigRows: newRows });
    formik?.setFieldValue('rows', newRows);
  };

  const renderOperationTypeColumn = (params?: any) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    return (
      <JobAutocomplete
        params={params}
        rows={rows}
        operationTypeList={operationTagValueList}
        handleChange={handleOperationTypeChange}
        cellName={appConstant?.jobOperationCellNames?.operationType}
      />
    );
  };

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

  const renderActionColumn = (params?: any) => {
    const row = rows?.find((row?) => row?.id === params?.id);
    return (
      <IconButton onClick={(e: MouseEvent<HTMLButtonElement>) => handleClick(e, row)}>
        <MoreVertIcon />
      </IconButton>
    );
  };

  const handleClick = (event: MouseEvent<HTMLButtonElement>, row?: any) => {
    setAnchorEl(event.currentTarget);
    setSelectedRowId(row?.id);
  };

  const handleClose = () => {
    setAnchorEl(null);
  };

  const handleDeleteRow = () => {
    setRows(rows.filter((row) => row?.id !== selectedRowId));
    handleClose();
  };

  const handleOpenPreviewQuery = () => {
    setShowQuery(true);
  };

  const handleClosePreviewQuery = () => {
    setShowQuery(false);
  };

  return (
    <Box p={3}>
      <Grid container spacing={2}>
        <Grid item xs={12} md={6}>
          <Autocomplete
            value={selectedAutocompleteValue}
            onChange={(event, newValue) => setSelectedAutocompleteValue(newValue)}
            options={operationTagValueList}
            getOptionLabel={(option) => option.name}
            renderInput={(params) => <TextField {...params} label={t('operationTag')} />}
          />
        </Grid>
        <Grid item xs={12} md={6}>
          <Button onClick={() => handleAddOperation()}>Add Operation</Button>
        </Grid>
      </Grid>
      <TableContainer component={Paper}>
        <Table>
          <TableHead>
            <TableRow>
              {operationConfigColumns?.map((column) => (
                <TableCell key={column.field}>{column.headerName}</TableCell>
              ))}
            </TableRow>
          </TableHead>
          <TableBody>
            {rows.length > 0 ? (
              rows?.map((row) => (
                <TableRow key={row.id}>
                  {operationConfigColumns?.map((column) => (
                    <TableCell key={column.field}>{column.renderCell ? column.renderCell({ value: row[column.field], id: row.id }) : row[column.field]}</TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={operationConfigColumns.length}>
                  <EmptyDatagridComponent />
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </TableContainer>
      <Menu anchorEl={anchorEl} open={Boolean(anchorEl)} onClose={handleClose}>
        <MenuItem onClick={handleDeleteRow}>Delete</MenuItem>
      </Menu>
      <PreviewQuery open={showQuery} onClose={handleClosePreviewQuery} />
    </Box>
  );
};

export default OperationConfiguration;
ilar functionality to the original `DataGrid` implementation but with MUI's `Table` component. Adjustments might be needed based on specific requirements or behavior.
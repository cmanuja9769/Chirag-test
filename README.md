const [connectionOptions, setConnectionOptions] = useState<{ CONNECTION_ID: string; CONNECTION_NAME: string }[]>([]);
  const [schemaOptions, setSchemaOptions] = useState<{ SCHEMA_NAME: string; ENTITY_NAME: string }[]>([]);
  const [objectOptions, setObjectOptions] = useState<{ SCHEMA_NAME: string; ENTITY_NAME: string }[]>([]);

[
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Schema Name",
        "operationTagValue": "PTCDH Landing Layer",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH Landing Layer"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Query",
        "operationTagValue": "RE-CON11",
        "operationType": "Hello",
        "operationTagValueGui": "RE-CON11"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Table Name",
        "operationTagValue": "LND_SAMPLES_TRANSACTION",
        "operationType": "Hello",
        "operationTagValueGui": "LND_SAMPLES_TRANSACTION"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Connection Id",
        "operationTagValue": "PTCDH-SNOWFLAKE-TEST",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH-SNOWFLAKE-TEST"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Schema Name",
        "operationTagValue": "PTCDH Landing Layer",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH Landing Layer"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Query",
        "operationTagValue": "RE-CON11",
        "operationType": "Hello",
        "operationTagValueGui": "RE-CON11"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Table Name",
        "operationTagValue": "LND_SAMPLES_TRANSACTION",
        "operationType": "Hello",
        "operationTagValueGui": "LND_SAMPLES_TRANSACTION"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Connection Id",
        "operationTagValue": "PTCDH-SNOWFLAKE-TEST",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH-SNOWFLAKE-TEST"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Schema Name",
        "operationTagValue": "PTCDH Landing Layer",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH Landing Layer"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Query",
        "operationTagValue": "RE-CON11",
        "operationType": "Hello",
        "operationTagValueGui": "RE-CON11"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Table Name",
        "operationTagValue": "LND_SAMPLES_TRANSACTION",
        "operationType": "Hello",
        "operationTagValueGui": "LND_SAMPLES_TRANSACTION"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Connection Id",
        "operationTagValue": "PTCDH-SNOWFLAKE-TEST",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH-SNOWFLAKE-TEST"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Schema Name",
        "operationTagValue": "PTCDH Landing Layer",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH Landing Layer"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Query",
        "operationTagValue": "RE-CON11",
        "operationType": "Hello",
        "operationTagValueGui": "RE-CON11"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Write Table Name",
        "operationTagValue": "LND_SAMPLES_TRANSACTION",
        "operationType": "Hello",
        "operationTagValueGui": "LND_SAMPLES_TRANSACTION"
    },
    {
        "operationSequence": "1",
        "operationName": "Query Ingestion",
        "operationTag": "Connection Id",
        "operationTagValue": "PTCDH-SNOWFLAKE-TEST",
        "operationType": "Hello",
        "operationTagValueGui": "PTCDH-SNOWFLAKE-TEST"
    }
]

<JobAutocomplete
        params={params}
        rows={rows}
        operationTagValueList={operationTagValueList}
        handleOperationTagValueChange={handleOperationTagValueChange}
        handleOperationTagValueOptionsChange={handleOperationTagValueOptionsChange}
        setOperationTagValueList={setOperationTagValueList}
        loadingTagValues={loadingTagValues[row.id]}
        handleFetchSchemaObjectChange={handleFetchSchemaObjectChange}
      />

JobAutoComplete.tsx
import React from 'react';
import { TextField, FormControl, Autocomplete, CircularProgress, InputAdornment } from '@mui/material';
import { useFormikContext } from 'formik';
import { OperationTagValueCellProps } from '../interface/operationConfig.interface';
import { appConstant } from '../../../utils/constants/appConstants';

const JobAutocomplete: React.FC<OperationTagValueCellProps> = ({
  params,
  operationTagValueList,
  handleOperationTagValueChange,
  handleOperationTagValueOptionsChange,
  setOperationTagValueList,
  loadingTagValues,
  handleFetchSchemaObjectChange
}) => {
  const { values, setFieldValue, touched, errors } = useFormikContext<any>();
  const operationTagValue = values?.rows[params?.id - 1]?.operationTagValue || '';
  const error = touched?.rows && touched?.rows[params?.id - 1] && errors?.rows && (errors?.rows[params?.id - 1] as any)?.operationTagValue;

  return (
    <FormControl className="tw-w-[20vw] tw-mt-5" error={Boolean(error)}>
      <Autocomplete
        ListboxProps={{ sx: { width: 'fit-content' } }}
        options={operationTagValueList?.map((option?: any) => option?.name) || []}
        value={operationTagValue}
        onFocus={() => {
          setOperationTagValueList([]);
          if (appConstant?.tagsForOnFocusApiCall?.includes(params?.row?.operationTag)) {
            handleFetchSchemaObjectChange(params);
          }
        }}
        onInputChange={(e, searchValue, reason) => {
          if (reason === appConstant?.input) {
            handleOperationTagValueOptionsChange(searchValue, params);
          }
        }}
        onChange={(e, selectedTagValue) => {
          const selectedTag = operationTagValueList?.find((option?) => option?.name === selectedTagValue);
          handleOperationTagValueChange(params?.id, selectedTagValue, selectedTag?.id);
          setFieldValue(`rows[${params?.id - 1}].operationTagValue`, selectedTagValue);
        }}
        renderInput={(params) => (
          <TextField
            {...params}
            error={Boolean(error)}
            InputProps={{
              ...params?.InputProps,
              endAdornment: (
                <>
                  {loadingTagValues && (
                    <InputAdornment position="end">
                      <CircularProgress size={20} />
                    </InputAdornment>
                  )}
                  {params?.InputProps?.endAdornment}
                </>
              )
            }}
          />
        )}
      />
    </FormControl>
  );
};

export default JobAutocomplete;

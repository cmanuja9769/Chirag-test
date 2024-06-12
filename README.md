import React, { createContext, ReactNode, useCallback, useContext, useEffect, useState } from 'react';
import * as Yup from 'yup';
import { DagData, DagContextProps, DagProviderProps } from '../../dag.interface';
import { createDagForm } from '../../../../utils/constants/appConstants';
import { buildValidationSchema } from '../../../../utils/validation';

const DagContext = createContext<DagContextProps>({
  dagData: null,
  updateDagData: () => {},
  step: 0,
  updateDagStage: () => {},
  triggerValidation: () => Promise.resolve(),
  setValidateDagDetailsForm: () => {},
  setValidateDagParamsForm: () => {},
  headers: [],
  charCount: 0,
  setCharCount: () => {}
});

export const DagProvider = ({ children }: DagProviderProps) => {
  const [dagData, setDagData] = useState({
    dag_name: '',
    description: '',
    active_flag: '',
    cron_schedule: '',
    max_active_runs: '',
    num_of_retries: '',
    timeout_in_mins: '',
    tags: '',
    email_on_success: '',
    email_on_failure: ''
  });

  const [validateDagDetailsForm, setValidateDagDetailsForm] = useState(() => () => Promise.resolve());
  const [validateDagParamsForm, setValidateDagParamsForm] = useState(() => () => Promise.resolve());
  const [headers, setHeaders] = useState([]);
  const [step, setStep] = useState(0);
  const [charCount, setCharCount] = useState(dagData?.description?.length);

  useEffect(() => {
    const fetchHeaders = async () => {
      const data = createDagForm;
      setHeaders(data.headers);
    };
    fetchHeaders();
  }, []);

  useEffect(() => {
    const dagDetailsHeaders = headers.filter((header) => header.SEQUENCE <= 3);
    const dagParamsHeaders = headers.filter((header) => header.SEQUENCE > 3);

    const dagDetailsValidationSchema = buildValidationSchema(dagDetailsHeaders);
    const dagParamsValidationSchema = buildValidationSchema(dagParamsHeaders);

    setValidateDagDetailsForm(() => async () => {
      try {
        await dagDetailsValidationSchema.validate(dagData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });

    setValidateDagParamsForm(() => async () => {
      try {
        await dagParamsValidationSchema.validate(dagData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [headers, dagData]);

  const updateDagData = (value: any) => {
    setDagData(value);
  };

  const updateDagStage = (value: any) => {
    setStep(value);
  };

  const triggerValidation = useCallback(async () => {
    try {
      await validateDagDetailsForm();
      await validateDagParamsForm();
      return Promise.resolve();
    } catch (errors) {
      return Promise.reject(errors);
    }
  }, [validateDagDetailsForm, validateDagParamsForm]);

  const context: DagContextProps = {
    dagData: dagData,
    updateDagData: updateDagData,
    step: step,
    updateDagStage: updateDagStage,
    triggerValidation: triggerValidation,
    setValidateDagDetailsForm: setValidateDagDetailsForm,
    setValidateDagParamsForm: setValidateDagParamsForm,
    headers: headers,
    charCount: charCount,
    setCharCount: setCharCount
  };

  return <DagContext.Provider value={context}>{children}</DagContext.Provider>;
};

export const useDagContext = () => useContext(DagContext);



import { useTranslation } from 'react-i18next';
import React, { useEffect, useState } from 'react';

import { Formik, Form } from 'formik';
import { useDagContext } from '../DagContext/index';
import { transformApiDataToFormFields } from '../../../../utils/formatObjectKeys';
import { Box, Grid, Typography } from '@mui/material';

import * as Yup from 'yup';
import DynamicComponent from '../../../DynamicForm';

const DagDetails: React.FC = () => {
  const { t } = useTranslation();
  const dagContext = useDagContext();
  const { headers, dagData, updateDagData, setValidateDagDetailsForm, setCharCount } = dagContext;
  const headersData = transformApiDataToFormFields(headers?.filter((header?) => header?.SEQUENCE <= 3));
  const [validationSchema, setValidationSchema] = useState(Yup?.object({}));

  useEffect(() => {
    const schema = Yup?.object()?.shape(
      headersData?.reduce((acc?, header?) => {
        if (header?.validation?.required) {
          acc[header?.name] = Yup?.string()?.required(`${header?.label} is required`);
        }
        return acc;
      }, {})
    );
    setValidationSchema(schema);
    setValidateDagDetailsForm(() => async () => {
      try {
        await schema?.validate(dagData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [headersData, dagData, setValidateDagDetailsForm]);

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue">
      <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
        {t('dag_details')}
      </Typography>
      <Formik
        initialValues={dagData}
        validationSchema={validationSchema}
        onSubmit={(values?) => {
          updateDagData(values);
        }}
        validateOnChange={false}
        validateOnBlur={true}
      >
        {({ values, handleBlur, setFieldValue }) => (
          <Form id="dag-details-form">
            <Box display="inline-flex" alignItems="center" padding={2} sx={{ '& .MuiFormControl-root': { width: '30vw' } }}>
              <Grid container spacing={3} direction="row" alignItems="center" justifyContent="center">
                {headersData
                  ?.sort((a, b) => a?.sequence - b?.sequence)
                  ?.map((header?) => (
                    <Grid item xs={12} sm={12} key={header?.name}>
                      <DynamicComponent
                        key={header?.name}
                        fieldData={header}
                        onChange={(value?) => {
                          if (header?.name === 'description' && value?.length <= header?.maxLength) {
                            setFieldValue(header?.name, value);
                            updateDagData((prevData?) => ({
                              ...prevData,
                              [header?.name]: value
                            }));
                            setCharCount(value?.length);
                          } else {
                            setFieldValue(header?.name, value);
                            updateDagData((prevData?) => ({
                              ...prevData,
                              [header?.name]: value
                            }));
                          }
                        }}
                        disabled={false}
                      />
                    </Grid>
                  ))}
              </Grid>
            </Box>
          </Form>
        )}
      </Formik>
    </Box>
  );
};

export default DagDetails;

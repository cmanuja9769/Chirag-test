import React, { createContext, ReactNode, useContext, useState } from 'react';
import { DagData, DagContextProps, DagProviderProps } from '../../dag.interface';
import { createDagForm } from '../../../../utils/constants/appConstants';

const DagContext = createContext<DagContextProps>({
  dagData: null,
  updateDagData: () => {},
  step: 0,
  updateDagStage: () => {},
  triggerValidation: () => Promise.resolve(),
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

  const updateDagData = (value: any) => {
    setDagData(value);
  };

  const updateDagStage = (value: any) => {
    setStep(value);
  };

  const triggerValidation = useCallback(async () => {
    // Validation logic moved to components
    return Promise.resolve();
  }, []);

  const context: DagContextProps = {
    dagData: dagData,
    updateDagData: updateDagData,
    step: step,
    updateDagStage: updateDagStage,
    triggerValidation: triggerValidation,
    headers: headers,
    charCount: charCount,
    setCharCount: setCharCount
  };

  return <DagContext.Provider value={context}>{children}</DagContext.Provider>;
};

export const useDagContext = () => useContext(DagContext);


import React, { useEffect, useState } from 'react';
import { useTranslation } from 'react-i18next';
import { Formik, Form } from 'formik';
import * as Yup from 'yup';
import { Box, Grid, Typography } from '@mui/material';
import { useDagContext } from '../DagContext';
import { transformApiDataToFormFields } from '../../../../utils/formatObjectKeys';
import DynamicComponent from '../../../DynamicForm';

const DagDetails: React.FC = () => {
  const { t } = useTranslation();
  const { headers, dagData, updateDagData, setCharCount } = useDagContext();
  const headersData = transformApiDataToFormFields(headers?.filter((header) => header?.SEQUENCE <= 3));

  const [validationSchema, setValidationSchema] = useState(Yup.object({}));

  useEffect(() => {
    const schema = Yup.object().shape(
      headersData.reduce((acc, header) => {
        if (header?.validation?.required) {
          acc[header?.name] = Yup.string().required(`${header?.label} is required`);
        }
        return acc;
      }, {})
    );
    setValidationSchema(schema);
  }, [headersData]);

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue">
      <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
        {t('dag_details')}
      </Typography>
      <Formik
        initialValues={dagData}
        validationSchema={validationSchema}
        onSubmit={(values) => {
          updateDagData(values);
        }}
        validateOnChange={false}
        validateOnBlur={true}
      >
        {({ values, handleBlur, setFieldValue, errors, touched }) => (
          <Form id="dag-details-form">
            <Box display="inline-flex" alignItems="center" padding={2} sx={{ '& .MuiFormControl-root': { width: '30vw' } }}>
              <Grid container spacing={3} direction="row" alignItems="center" justifyContent="center">
                {headersData
                  ?.sort((a, b) => a?.sequence - b?.sequence)
                  ?.map((header) => (
                    <Grid item xs={12} sm={12} key={header?.name}>
                      <DynamicComponent
                        key={header?.name}
                        fieldData={header}
                        value={values[header?.name]}
                        onChange={(value) => {
                          if (header?.name === 'description' && value?.length <= header?.maxLength) {
                            setFieldValue(header?.name, value);
                            updateDagData((prevData) => ({
                              ...prevData,
                              [header?.name]: value
                            }));
                            setCharCount(value?.length);
                          } else {
                            setFieldValue(header?.name, value);
                            updateDagData((prevData) => ({
                              ...prevData,
                              [header?.name]: value
                            }));
                          }
                        }}
                        onBlur={handleBlur}
                        error={errors[header?.name]}
                        touched={touched[header?.name]}
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


import React, { useEffect, useState } from 'react';
import { useTranslation } from 'react-i18next';
import { Formik, Form } from 'formik';
import { useDagContext } from '../DagContext';
import DynamicComponent from '../../../DynamicForm';
import { Box, Grid, Typography, TextField, Chip } from '@mui/material';
import * as Yup from 'yup';

const DagParams: React.FC = () => {
  const { t } = useTranslation();
  const { headers, dagData, updateDagData } = useDagContext();

  const headersData = headers.filter((header) => header.SEQUENCE !== 4);

  const [validationSchema, setValidationSchema] = useState(Yup.object({}));

  useEffect(() => {
    const schema = Yup.object().shape(
      headersData.reduce((acc, header) => {
        if (header.validation?.required) {
          acc[header.name] = Yup.string().required(`${header.label} is required`);
        }
        return acc;
      }, {})
    );
    setValidationSchema(schema);
  }, [headersData]);

  return (
    <Box className="tw-mt-5 tw-block tw-pb-2 tw-border-solid tw-border-2 tw-border-pfizerBlue">
      <Typography variant="h6" component="h6" padding={1} className="tw-bg-pfizerBlue tw-text-white tw-font-bold">
        {t('dag_params')}
      </Typography>
      <Formik
        initialValues={dagData}
        validationSchema={validationSchema}
        onSubmit={(values) => {
          updateDagData(values);
        }}
        validateOnChange={false}
        validateOnBlur={true}
      >
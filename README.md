import React, { createContext, useState, useContext } from 'react';
import { DagContextProps, DagProviderProps } from '../../dag.interface';
import { createDagForm } from '../../../../utils/constants/appConstants';

const DagContext = createContext<DagContextProps>({
  dagData: null,
  updateDagData: () => {},
  step: 0,
  updateDagStage: () => {},
  triggerValidation: () => Promise.resolve(),
  headers: [],
  charCount: 0,
  setCharCount: () => {},
  setDagDetailsValidation: () => {},
  setDagParamsValidation: () => {},
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
  const [validateDagDetails, setDagDetailsValidation] = useState(() => () => Promise.resolve());
  const [validateDagParams, setDagParamsValidation] = useState(() => () => Promise.resolve());

  const fetchHeaders = async () => {
    const data = createDagForm;
    setHeaders(data.headers);
  };

  useEffect(() => {
    fetchHeaders();
  }, []);

  const updateDagData = (value: any) => {
    setDagData(value);
  };

  const updateDagStage = (value: any) => {
    setStep(value);
  };

  const triggerValidation = async () => {
    try {
      await validateDagDetails();
      await validateDagParams();
      return Promise.resolve();
    } catch (errors) {
      return Promise.reject(errors);
    }
  };

  const context: DagContextProps = {
    dagData: dagData,
    updateDagData: updateDagData,
    step: step,
    updateDagStage: updateDagStage,
    triggerValidation: triggerValidation,
    headers: headers,
    charCount: charCount,
    setCharCount: setCharCount,
    setDagDetailsValidation: setDagDetailsValidation,
    setDagParamsValidation: setDagParamsValidation,
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
  const { headers, dagData, updateDagData, setCharCount, setDagDetailsValidation } = useDagContext();
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
    setDagDetailsValidation(() => async () => {
      try {
        await schema.validate(dagData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [headersData, dagData, setDagDetailsValidation]);


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
    setDagParamsValidation(() => async () => {
      try {
        await schema.validate(dagData, { abortEarly: false });
        return Promise.resolve();
      } catch (errors) {
        return Promise.reject(errors);
      }
    });
  }, [headersData, dagData, setDagParamsValidation]);
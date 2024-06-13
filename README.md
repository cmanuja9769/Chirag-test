// useValidation.ts
import { useCallback } from 'react';
import { FormikHelpers } from 'formik';

type ValidationFunction = (values: any, formikHelpers: FormikHelpers<any>) => Promise<void>;

const useValidation = (validateDagDetails: ValidationFunction, validateDagParams: ValidationFunction) => {
  const triggerValidation = useCallback(async (formikHelpers: FormikHelpers<any>, initialValues: any) => {
    try {
      await validateDagDetails(formikHelpers.values, formikHelpers);
      await validateDagParams(formikHelpers.values, formikHelpers);
    } catch (errors) {
      throw errors;
    }
  }, [validateDagDetails, validateDagParams]);

  return { triggerValidation };
};

export default useValidation;

const formikContext = useFormikContext(); // Get Formik context

  const handleNextClick = async () => {
    try {
      await triggerValidation(formikContext, dagContext.dagData); // Pass dagData as initialValues
      
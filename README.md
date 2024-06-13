// useValidation.ts
import { useCallback } from 'react';
import { FormikHelpers } from 'formik';

type ValidationFunction = (values: any, formikHelpers: FormikHelpers<any>) => Promise<void>;

const useValidation = (validateDagDetails: ValidationFunction, validateDagParams: ValidationFunction) => {
  const triggerValidation = useCallback(async (values: any, formikHelpers: FormikHelpers<any>) => {
    try {
      await validateDagDetails(values, formikHelpers);
      await validateDagParams(values, formikHelpers);
    } catch (errors) {
      throw errors;
    }
  }, [validateDagDetails, validateDagParams]);

  return { triggerValidation };
};

export default useValidation;


const formikContext = useFormikContext();

  const handleNextClick = async () => {
    try {
      await triggerValidation(formikContext.values, {
        setTouched: formikContext.setTouched,
        initialValues: formikContext.initialValues ?? {}
      });

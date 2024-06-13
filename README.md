import { useCallback } from 'react';
import { FormikHelpers } from 'formik';

const useValidation = (validateDagDetails: () => Promise<void>, validateDagParams: () => Promise<void>) => {
  const triggerValidation = useCallback(async (formikHelpers: FormikHelpers<any>) => {
    try {
      // Mark all fields as touched
      formikHelpers.setTouched(
        Object.keys(formikHelpers.initialValues).reduce((acc, field) => {
          acc[field] = true;
          return acc;
        }, {} as Record<string, boolean>)
      );
      // Run the validations
      await validateDagDetails();
      await validateDagParams();
      return Promise.resolve();
    } catch (errors) {
      return Promise.reject(errors);
    }
  }, [validateDagDetails, validateDagParams]);

  return { triggerValidation };
};

export default useValidation;
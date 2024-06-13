import { useCallback } from 'react';
import { FormikHelpers, FormikValues } from 'formik';

const useValidation = (validateDagDetails: () => Promise<void>, validateDagParams: () => Promise<void>) => {
  const triggerValidation = useCallback(
    async (formikHelpers: FormikHelpers<FormikValues>, initialValues: FormikValues) => {
      try {
        // Mark all fields as touched
        formikHelpers.setTouched(
          Object.keys(initialValues).reduce((acc, field) => {
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
    },
    [validateDagDetails, validateDagParams]
  );

  return { triggerValidation };
};

export default useValidation;
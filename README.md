import React from 'react';
import { Button } from '@mui/material';
import { FormikProps } from 'formik';

interface SubmitButtonProps {
  formRefs: React.RefObject<FormikProps<any>>[];
}

const SubmitButton: React.FC<SubmitButtonProps> = ({ formRefs }) => {
  const handleClick = () => {
    formRefs.forEach((formRef) => {
      if (formRef.current) {
        formRef.current.submitForm().then(() => {
          console.log(`${formRef.current?.props.initialValues.constructor.name} submitted successfully`);
        }).catch((error) => {
          console.error(`Error submitting ${formRef.current?.props.initialValues.constructor.name}:`, error);
        });
      } else {
        console.error(`${formRef.current?.props.initialValues.constructor.name} ref is not yet initialized`);
      }
    });
  };

  return (
    <Button
      type="button"
      variant="contained"
      color="primary"
      onClick={handleClick}
    >
      Submit All Forms
    </Button>
  );
};

export default SubmitButton;

import React, { useRef } from 'react';
import MyForm, { MyFormValues } from './MyForm';
import TyForm, { TyFormValues } from './TyForm';
import SubmitButton from './SubmitButton';
import { FormikProps } from 'formik';

const App: React.FC = () => {
  const myFormRef = useRef<FormikProps<MyFormValues>>(null);
  const tyFormRef = useRef<FormikProps<TyFormValues>>(null);

  const formRefs = [myFormRef, tyFormRef];

  return (
    <div>
      <MyForm formRef={myFormRef} />
      <TyForm formRef={tyFormRef} />
      <SubmitButton formRefs={formRefs} />
    </div>
  );
};

export default App;





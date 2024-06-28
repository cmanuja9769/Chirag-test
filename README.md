import { createContext, useContext, useState, ReactNode } from 'react';
import { JobContextProps, JobProviderProps, JobData } from '../components/Jobs/job.interface';

const JobContext = createContext<JobContextProps>({
  jobData: null,
  step: 0,
  headers: [],
  charCount: 0,
  updateJobData: () => {},
  updateStage: () => {},
  updateHeaders: () => {},
  setCharCount: () => {},
  validateJobDetails: () => Promise.resolve(),
  validateJobSource: () => Promise.resolve(),
  validateJobTarget: () => Promise.resolve(),
  setJobDetailsValidation: () => {},
  setJobSourceValidation: () => {},
  setJobTargetValidation: () => {},
  isNexButtonClicked: false,
  handleNextClick: () => {}
});

export const JobProvider = ({ children }: JobProviderProps) => {
  const [jobData, setJobData] = useState<JobData>({
    job_name: '',
    description: '',
    job_type: '',
    source_conName: '',
    source_schema: '',
    source_object: '',
    target_conName: '',
    target_object: '',
    target_schema: '',
    selectedOption: '',
    checkboxStates: {},
    previewData: {}
  });

  const [headers, setHeaders] = useState([]);
  const [step, setStep] = useState(0);
  const [charCount, setCharCount] = useState(256);
  const [validateJobDetails, setJobDetailsValidation] = useState(() => () => Promise.resolve());
  const [validateJobSource, setJobSourceValidation] = useState(() => () => Promise.resolve());
  const [validateJobTarget, setJobTargetValidation] = useState(() => () => Promise.resolve());
  const [isNexButtonClicked, setNextButtonClicked] = useState(false);

  const updateJobData = (value: Partial<JobData>) => {
    setJobData((prevJobData) => ({
      ...prevJobData,
      ...value
    }));
  };

  const updateStage = (value: number) => {
    setStep(value);
  };

  const updateHeaders = (value: any[]) => {
    setHeaders(value);
  };

  const handleNextClick = (value: boolean) => {
    setNextButtonClicked(value);
  };

  const updateCheckboxState = (checkbox: string) => {
    setJobData((prevState) => ({
      ...prevState,
      checkboxStates: {
        ...prevState.checkboxStates,
        [checkbox]: !prevState.checkboxStates[checkbox]
      }
    }));
  };

  const updateSelectedOption = (option: string) => {
    setJobData((prevState) => ({
      ...prevState,
      selectedOption: option,
      checkboxStates: {}
    }));
  };

  const updatePreviewData = (checkbox: string, data: string) => {
    setJobData((prevState) => ({
      ...prevState,
      previewData: {
        ...prevState.previewData,
        [checkbox]: data
      }
    }));
  };

  const context: JobContextProps = {
    jobData,
    step,
    headers,
    charCount,
    updateJobData,
    updateStage,
    updateHeaders,
    setCharCount,
    validateJobDetails,
    validateJobSource,
    validateJobTarget,
    setJobDetailsValidation,
    setJobSourceValidation,
    setJobTargetValidation,
    isNexButtonClicked,
    handleNextClick,
    updateCheckboxState,
    updateSelectedOption,
    updatePreviewData
  };

  return <JobContext.Provider value={context}>{children}</JobContext.Provider>;
};

export const useJobContext = () => useContext(JobContext);
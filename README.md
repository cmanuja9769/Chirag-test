import { createContext, useContext, useState, ReactNode } from 'react';
import { JobContextProps, JobProviderProps } from '../components/Jobs/job.interface';

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
  const [jobData, setJobData] = useState({
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

  const updateJobData = (value: any) => {
    setJobData(value);
  };

  const updateStage = (value: any) => {
    setStep(value);
  };

  const updateHeaders = (value: any) => {
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
        [checkbox]: !prevState.checkboxStates[checkbox],
      },
    }));
  };

  const updateSelectedOption = (option: string) => {
    setJobData((prevState) => ({
      ...prevState,
      selectedOption: option,
      checkboxStates: {},
    }));
  };

  const updatePreviewData = (checkbox: string, data: string) => {
    setJobData((prevState) => ({
      ...prevState,
      previewData: {
        ...prevState.previewData,
        [checkbox]: data,
      },
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
    updatePreviewData,
  };

  return <JobContext.Provider value={context}>{children}</JobContext.Provider>;
};

export const useJobContext = () => useContext(JobContext);

import { ReactNode } from 'react';

export interface JobDataProps {
  id?: number;
  name?: string;
  tags?: string[];
  createdtime?: string;
  description?: string;
  status?: string;
}

export interface JobData {
  job_name?: string;
  description?: string;
  job_type?: string;
  source_conName?: string;
  source_schema?: string;
  source_object?: string;
  target_conName?: string;
  target_object?: string;
  target_schema?: string;
  selectedOption?: string;
  checkboxStates?: { [key: string]: boolean };
  previewData?: { [key: string]: string };
}

export interface JobContextProps {
  jobData?: JobData;
  step?: number;
  headers: any[];
  charCount: number;
  updateJobData?: React.Dispatch<React.SetStateAction<JobData>>;
  updateStage?: React.Dispatch<React.SetStateAction<number>>;
  setValidateJobDetailsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setValidateJobParamsForm?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  updateHeaders?: React.Dispatch<React.SetStateAction<any[]>>;
  setCharCount: React.Dispatch<React.SetStateAction<number>>;
  validateJobDetails?: () => Promise<void>;
  validateJobSource?: () => Promise<void>;
  validateJobTarget?: () => Promise<void>;
  setJobDetailsValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobSourceValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  setJobTargetValidation?: React.Dispatch<React.SetStateAction<() => Promise<void>>>;
  isNexButtonClicked?: boolean;
  handleNextClick?: React.Dispatch<React.SetStateAction<boolean>>;
  updateCheckboxState?: (checkbox: string) => void;
  updateSelectedOption?: (option: string) => void;
  updatePreviewData?: (checkbox: string, data: string) => void;
}

export interface JobProviderProps {
  children?: ReactNode;
}

export interface JobHeaderProps {
  onClose?: () => void;
  step?: number;
  onBack?: () => void;
}
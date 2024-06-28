import { createContext, useContext, useState, ReactNode, useEffect } from 'react';
import { JobContextProps, JobProviderProps, JobData } from '../components/Jobs/job.interface';

const JobContext = createContext<JobContextProps>({
  jobData: {},
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

  const [headers, setHeaders] = useState<any[]>([]);
  const [step, setStep] = useState<number>(0);
  const [charCount, setCharCount] = useState<number>(256);
  const [validateJobDetails, setJobDetailsValidation] = useState<() => Promise<void>>(() => () => Promise.resolve());
  const [validateJobSource, setJobSourceValidation] = useState<() => Promise<void>>(() => () => Promise.resolve());
  const [validateJobTarget, setJobTargetValidation] = useState<() => Promise<void>>(() => () => Promise.resolve());
  const [isNexButtonClicked, setNextButtonClicked] = useState<boolean>(false);

  const updateJobData = (value: JobData) => {
    setJobData(value);
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
  };

  useEffect(() => {
    if (isNexButtonClicked) {
      validateJobTarget().then(() => {
        // Do something after validation success if needed
        setNextButtonClicked(false);
      }).catch((errors) => {
        console.error(errors);
        setNextButtonClicked(false); // Reset button click state on validation error
      });
    }
  }, [isNexButtonClicked, validateJobTarget]);

  return <JobContext.Provider value={context}>{children}</JobContext.Provider>;
};

export const useJobContext = () => useContext(JobContext);


import { ReactNode } from 'react';

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
  updateJobData?: (value: JobData) => void;
  updateStage?: (value: number) => void;
  updateHeaders?: (value: any[]) => void;
  setCharCount: (value: number) => void;
  validateJobDetails?: () => Promise<void>;
  validateJobSource?: () => Promise<void>;
  validateJobTarget?: () => Promise<void>;
  setJobDetailsValidation?: (fn: () => Promise<void>) => void;
  setJobSourceValidation?: (fn: () => Promise<void>) => void;
  setJobTargetValidation?: (fn: () => Promise<void>) => void;
  isNexButtonClicked?: boolean;
  handleNextClick?: (value: boolean) => void;
}

export interface JobProviderProps {
  children?: ReactNode;
}

export interface JobHeaderProps {
  onClose?: () => void;
  step?: number;
  onBack?: () => void;
}
describe('CollateralTaskStepsMapperService - mapDocData', () => {
  let service: any;
  let mockUtils: any;
  let mockHydrateService: any;
  let mockLoggerService: any;
  let mockContextService: any;

  beforeEach(() => {
    mockUtils = { isEmpty: jasmine.createSpy() };
    mockHydrateService = {
      getObjectValue: jasmine.createSpy(),
      hydrateData: jasmine.createSpy()
    };
    mockLoggerService = { log: jasmine.createSpy() };
    mockContextService = { getUserContext: () => ({ userName: 'mockUser' }) };

    service = new CollateralTaskStepsMapperService(
      null, null, null, null,
      mockLoggerService,
      null,
      mockContextService,
      null,
      mockHydrateService,
      null,
      null
    );
    service.utils = mockUtils;
    service.taskInfoGetResponse = {}; // dummy response
  });

  it('should map document info when data is present', () => {
    const docContainer = {
      dataMapModel: true,
      dataMapModel: [
        {
          name: 'docName',
          type: 'DOCLIST'
        }
      ],
      documentsInfo: {}
    };

    const getModelObj = [{ field1: 'val1', documentUrl: 'url' }];
    const hydratedData = [{ field1: 'x', documentUrl: 'y' }];

    mockUtils.isEmpty.and.returnValues(false, false);
    mockHydrateService.getObjectValue.and.returnValue(getModelObj);
    mockHydrateService.hydrateData.and.returnValue(hydratedData);

    service.mapDocData(docContainer);

    expect(docContainer.documentsInfo['documentListInfo']).toBeDefined();
  });

  it('should log error when hydrated data is empty', () => {
    const docContainer = {
      dataMapModel: true,
      dataMapModel: [
        {
          name: 'docName',
          type: 'DOCLIST'
        }
      ],
      documentsInfo: {}
    };

    mockUtils.isEmpty.and.returnValue(false); // first check for isEmpty on dataMapModel
    mockHydrateService.getObjectValue.and.returnValue(null);
    mockHydrateService.hydrateData.and.returnValue([]);

    service.mapDocData(docContainer);

    expect(mockLoggerService.log).toHaveBeenCalledWith(
      'error',
      'Collateraltask web UI ::',
      { key: 'error', value: 'Document Data is not available from hydration data' }
    );
  });
});

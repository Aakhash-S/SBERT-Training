describe('CollateralTaskStepsMapperService', () => {
  let service: CollateralTaskStepsMapperService;

  beforeEach(() => {
    service = new CollateralTaskStepsMapperService();
  });

  describe('updateDocumentContainer', () => {
    it('should call mapDocData when docContainer is not empty', () => {
      service.taskProfileConfig = { docContainer: { test: 'data' } };
      spyOn(service, 'mapDocData');

      service.updateDocumentContainer();

      expect(service.mapDocData).toHaveBeenCalledWith(service.taskProfileConfig.docContainer);
    });

    it('should not call mapDocData when docContainer is empty', () => {
      service.taskProfileConfig = { docContainer: {} };
      spyOn(service, 'mapDocData');

      service.updateDocumentContainer();

      expect(service.mapDocData).not.toHaveBeenCalled();
    });
  });
});

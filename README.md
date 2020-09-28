# assignment
struct UserPostData {
    var id: String?
    var userSub: String?
    var fileName: String?
    var price: Int?
    var userPostedImage: UIImage?
    var description: String?
    var dateUploaded: String?
}


class ResultsTableViewController: UITableViewController, priceLowToHigh, priceHightoLow {
    func sortPriceLowToHigh(sort: String) {
        print(sort)
         print(data)
            data.sort(by: { $0.price > $1.price}) // This is where I get the error 
    }
    func sortPriceHighToLow(sort: String) {
        print(sort)
    }
  
    var data: [UserPostData] = []
    var userSubID = String()
    let activityIndicator = UIActivityIndicatorView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setView()
        getData()
    }
    override func numberOfSections(in tableView: UITableView) -> Int {
        // #warning Incomplete implementation, return the number of sections
        return data.count
    }
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 1
    }
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = self.tableView.dequeueReusableCell(withIdentifier: "Results") as! CellResults
        cell.image = data[indexPath.section].userPostedImage
        cell.price = data[indexPath.section].price
        cell.layoutSubviews()
        self.tableView.rowHeight = UITableView.automaticDimension
        return cell
    }
    func setView(){
        let sortButton = UIBarButtonItem(title: "Sort", style: .plain, target: self, action: #selector(sortPressed))
        navigationItem.rightBarButtonItem = sortButton
    }
    func getData(){
        activityIndicator.hidesWhenStopped = true
        activityIndicator.translatesAutoresizingMaskIntoConstraints = false
        activityIndicator.style = .large
        activityIndicator.startAnimating()
        tableView.addSubview(activityIndicator)
        NSLayoutConstraint.activate([activityIndicator.centerXAnchor.constraint(equalTo: tableView.safeAreaLayoutGuide.centerXAnchor),
                                     activityIndicator.centerYAnchor.constraint(equalTo: tableView.safeAreaLayoutGuide.centerYAnchor)])
        userSubID = AWSMobileClient.default().userSub!
        
        let post = Post.keys
        let predicate = post.search == search
        _ = Amplify.API.query(request: .list(SellerPost.self, where: predicate)) { event in
            switch event {
            case .success(let result):
                switch result {
                case .success(let posts):
                    DispatchQueue.main.async {
            
                        self.tableView.reloadData()
     
                        for element in posts{
                            _ = Amplify.Storage.downloadData(key: element.filename,progressListener: { progress in
                                print("Progress: \(progress)")},
                                                             resultListener: { (event) in
                                                                switch event {
                                                                case let .success(data):
                                                                    DispatchQueue.main.async {
                                                                        
                                                                        
                                                                        self.data.append(UserPostData.init(id: element.id, userSub: element.userSub, fileName: element.filename, price: element.price, userPostedImage: UIImage(data: data), description: element.description,dateUploaded: element.dateUploaded))
                                                                        self.tableView.reloadData()
                                                                        self.activityIndicator.stopAnimating()
                                                                    }
                                                                    print("Completed: \(data)")
                                                                case let .failure(storageError):
                                                                    print("Failed: \(storageError.errorDescription). \(storageError.recoverySuggestion)")
                                                                }
                            })
 
                        }
                        
                    }
                case .failure(let error):
                    print("Got failed result with \(error.errorDescription)")
                }
            case .failure(let error):
                print("Got failed event with error \(error)")
            }
        }
        
        self.tableView.register(CellResults.self, forCellReuseIdentifier: "Results")
        
    }
 
    @objc func sortPressed(){
        print("sort clicked")
        let sTVC = SortViewController()
        sTVC.priceLowToHighDelegate = self
        sTVC.priceHighToLowDelegate = self
        sTVC.modalPresentationStyle = .custom
        self.present(sTVC, animated: true, completion: nil)
    }
}

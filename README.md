private static List<OptimalNumber> getOptimalNumber(int[] structures, int seniorCapacity, int juniorCapacity) {

        List<OptimalNumber> optimalNumbers = new ArrayList<>();
        for (int rooms : structures) {
            OptimalNumber optimalNumber = getOptimalNumber(rooms, seniorCapacity, juniorCapacity);
            optimalNumbers.add(optimalNumber);
        }

        return optimalNumbers;
    }

    private static OptimalNumber getOptimalNumber(int rooms, int seniorCapacity, int juniorCapacity) {
        if (lessThanSeniorCapacity(rooms, seniorCapacity)) {
            return new OptimalNumber(1, 0);
        }
        rooms = rooms - seniorCapacity;
        List<OptimalNumber> list = new ArrayList<>();
        OptimalNumber remainderForAllSeniors = remainderForAllSeniors(rooms, seniorCapacity);
        OptimalNumber remainderForAllJuniors = remainderForAllJuniors(rooms, juniorCapacity);
        Optional<OptimalNumber> remainderForSeniorsAndJuniors = remainderForSeniorsAndJuniors(rooms, seniorCapacity, juniorCapacity);

        list.add(remainderForAllJuniors);
        list.add(remainderForAllSeniors);
        if(remainderForSeniorsAndJuniors.isPresent()){
            list.add(remainderForSeniorsAndJuniors.get());
        }
        OptimalNumber optimalNumber = list.stream().sorted().findFirst().get();
        optimalNumber.setSeniors(optimalNumber.getSeniors()+1);
        return  optimalNumber;
    }

    private static Optional<OptimalNumber> remainderForSeniorsAndJuniors(int rooms, int seniorCapacity, int juniorCapacity) {
        List<OptimalNumber> list = new ArrayList<>();
        int couter = 0;
        while(rooms >= seniorCapacity){
            rooms = rooms - seniorCapacity;
            couter++;
            int division = rooms/juniorCapacity;
            int remainder = rooms % juniorCapacity;
            if(remainder > 0){
                division++;
            }
            OptimalNumber optimalNumber = new OptimalNumber(couter,division);
            optimalNumber.setRemainder(juniorCapacity - remainder);
            list.add(optimalNumber);
        }
        return list.stream().sorted().findFirst();
    }

    private static OptimalNumber remainderForAllJuniors(int rooms, int juniorCapacity) {
        int division = rooms / juniorCapacity;
        int remainder = rooms % juniorCapacity;
        if (remainder > 0) {
            division ++;
        }
        OptimalNumber optimalNumber = new OptimalNumber();
        optimalNumber.setJuniors(division);
        optimalNumber.setSeniors(0);
        optimalNumber.setRemainder(juniorCapacity - remainder);
        return optimalNumber;
    }

    private static OptimalNumber remainderForAllSeniors(int rooms, int seniorCapacity) {
        int division = rooms / seniorCapacity;
        int remainder = rooms % seniorCapacity;
        if (remainder > 0) {
            division++;
        }
        OptimalNumber optimalNumber = new OptimalNumber();
        optimalNumber.setSeniors(division);
        optimalNumber.setJuniors(0);
        optimalNumber.setRemainder(seniorCapacity - remainder);
        return optimalNumber;
    }

    private static boolean lessThanSeniorCapacity(int rooms, int seniorCapacity) {
        return rooms <= seniorCapacity;
    }
